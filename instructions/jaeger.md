## "Dream Within a Dream"

As Cobb and Arthur in *Inception*, let's perform a *"Trace Within a Trace" Strategy* called **Distributed Tracing** using Red Hat Openshift Container Platform to infiltrate the application traces and extract valuable information to solve the issues

![Inception]({% image_path inception.jpg %}){:width="300px"}

#### What is Jaeger?

![Jaeger]({% image_path jaeger-logo.png %}){:width="300px"}

[Jaeger Tracing](https://www.jaegertracing.io), inspired by Dapper and OpenZipkin, is a distributed tracing system released as open source by Uber Technologies. It is used for monitoring and troubleshooting microservices-based distributed systems, including:

* Distributed context propagation
* Distributed transaction monitoring
* Root cause analysis
* Service dependency analysis
* Performance / latency optimization

[Kiali](https://www.kiali.io) includes [Jaeger Tracing](https://www.jaegertracing.io) to provide distributed tracing out of the box.

#### What are you hidding, Mr/Mrs *Application*?

From the [Kiali Console]({{ KIALI_URL }}), `click on the Distributed Tracing` link in the left navigation and enter the following configuration:

 * Select a Namespace: **{{COOLSTORE_PROJECT}}**
 * Select a Service: **gateway**
 * Then click on the **magnifying glass** on the right

![Jaeger - Traces View]({% image_path jaeger-trace-2spans-view.png %}){:width="700px"}

By default, Service Mesh automatically sends collected tracing data to Jaeger, so that we are able to only see individual span (one-to-one service call).

* 1 span for ***Gateway Service*** -> ***Catalog Service***
* 7 spans for ***Gateway Service*** -> ***Inventory Service***

As you have called several times the ***Gateway Service*** through the Web UI, you find much more than 8 spans in Jaeger and you cannot easily observe the entire trace for an end-to-end request.

#### Enabling Distributed Context Propagation

**Distributed Tracing** involves propagating the tracing context from service to service by sending certain incoming HTTP headers downstream to outbound requests. To do this, services need some hints to tie together the entire trace. They need to propagate the appropriate HTTP headers so that when the proxies send span information, the spans can be correlated correctly into a single trace.

Let's enable Distributed Context Propagation from the ***Gateway Service***.

First, you are going to intercept the following header creating by Service Mesh in order to add them into the outbound requests:

 * x-request-id
 * x-b3-traceid
 * x-b3-spanid
 * x-b3-parentspanid
 * x-b3-sampled
 * x-b3-flags
 * x-ot-span-context

In CodeReady Workspaces, create a new class ***TracingInterceptor*** class in the ***com.redhat.cloudnative.gateway*** package in the **src** directory as following

~~~java
package com.redhat.cloudnative.gateway;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.function.Function;
import java.util.stream.Collectors;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import io.vertx.core.Handler;
import io.vertx.ext.web.client.impl.WebClientInternal;
import io.vertx.rxjava.ext.web.RoutingContext;
import io.vertx.rxjava.ext.web.client.WebClient;

public class TracingInterceptor {
    private static final Logger LOG = LoggerFactory.getLogger(TracingInterceptor.class);
    
    private static final List<String> FORWARDED_HEADER_NAMES = Arrays.asList(
        "x-request-id",
        "x-b3-traceid",
        "x-b3-spanid",
        "x-b3-parentspanid",
        "x-b3-sampled",
        "x-b3-flags",
        "x-ot-span-context"
    );

    private static final String X_TRACING_HEADERS = "X-Tracing-Headers";

    private TracingInterceptor() {
        // Avoid direct instantiation.
    }

    static Handler<RoutingContext> create() {
        return rc -> {
            Set<String> names = rc.request().headers().names();
            Map<String, List<String>> headers = names.stream()
                .map(String::toLowerCase)
                .filter(FORWARDED_HEADER_NAMES::contains)
                .collect(Collectors.toMap(
                    Function.identity(),
                    h -> Collections.singletonList(rc.request().getHeader(h))
                ));
            rc.put(X_TRACING_HEADERS, headers);
            rc.next();
        };
    }
    
    static WebClient propagate(WebClient client, RoutingContext rc) {
        WebClientInternal delegate = (WebClientInternal) client.getDelegate();
        delegate.addInterceptor(ctx -> {
            Map<String, List<String>> headers = rc.get(X_TRACING_HEADERS);
            if (headers != null) {
                LOG.info("Propagating header: {}", headers);
                headers.forEach((s, l) -> l.forEach(v -> ctx.request().putHeader(s, v)));
            }
            ctx.next();
        });
        return client;
    }
}
~~~

Then, route all traffic into the ***TracingInterceptor*** handler by replacing the comment ***// Enable TraceInterceptor handler here*** in the ***start()*** method with the following code:

~~~java
router.route()
    .order(-1)
    .handler(TracingInterceptor.create());
~~~

Finally, propagate the headers from the incoming request ***Gateway Service*** to any outgoing requests ***Catalog Service*** and ***Inventory Service*** using the ***propagate()*** method from ***TracingInterceptor*** class when calling outgoing services in the ***products()*** method.
 
<pre>
    <code>
        private void products(RoutingContext rc) {
            [...]
            <del>catalog.get("/api/catalog")</del>
            TracingInterceptor.propagate(catalog, rc).get("/api/catalog")
            [...]
            <del>inventory.get("/api/inventory/" + product.getString("itemId"))</del>
            TracingInterceptor.propagate(inventory, rc).get("/api/inventory/" + product.getString("itemId"))
            [...]
        }
    </code>
</pre>

Now check your modifiy then push the new version of the source code to OpenShift.

~~~shell
$ mvn clean package -f /projects/labs/gateway-vertx
$ oc start-build gateway-s2i --from-dir /projects/labs/gateway-vertx/ --follow
~~~

`Go back to Distributed Tracing` menu from [Kiali Console]({{ KIALI_URL }}) and see the result.
Now you have the aggreate trace for one request and it is much more better.
On the left hand side, you have information like the duration.
One call takes more than 400ms which you could judge as *normal* but ...

Let’s click on a trace title bar.

![Jaeger - Trace Detail View]({% image_path jaeger-trace-delay-detail-view.png %}){:width="700px"}

Interesting... The major part of a call is consuming by the ***Catalog Service***.
So let's have a look on its code. 
Go through the **catalog-spring-boot** source code and find the following piece of code.

~~~java
public List<Product> getAll() {
    Spliterator<Product> products = repository.findAll().spliterator();
    Random random = new Random();

    List<Product> result = new ArrayList<Product>();
    products.forEachRemaining(product -> {
        Class<Product> clazz = Product.class;
        if (clazz.isInstance(product)){
            try {
                Thread.sleep(random.nextInt(10) * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        result.add(product);
    });
    return result;
}
~~~

And yes, this burns your eyes, right?! Basically nobody could understand what the developer attempted to achieve but we do not have the time for that.
This piece of code is a part of the `getAll()` method which returns the list of all products from the database. 
As you are an expert of Java 8, you are about to create a masterpiece by both simplifying the code and increasing performance. 

Replace the content of the `getAll()` method as following:

~~~java
    public List<Product> getAll() {
        Spliterator<Product> products = repository.findAll().spliterator();
        return StreamSupport.stream(products, false).collect(Collectors.toList());
    }
~~~

> Do not forget to import the missing packages.

Now let's push the new version of the source code.

~~~shell
$ oc start-build catalog-s2i --from-dir /projects/labs/catalog-spring-boot/ --follow
~~~

![Jaeger - Trace Detail View]({% image_path jaeger-trace-fixed-detail-view.png %}){:width="700px"}

Just wonderful! You reduce the response time by a factor of 5!! You should be proud!!

**CONGRATULATIONS!!!** You make it but **is the spinning top stopped or not at the end?**

![Inception - Spinning Top]({% image_path spinningtop.jpg %}){:width="300px"}

We will never know and now, it is time to go deeper again!!
