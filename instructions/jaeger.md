## "Inception" Quotes

Context BlaBlaBla

#### What is Jaeger?

[Jaeger Tracing](https://www.jaegertracing.io), inspired by Dapper and OpenZipkin, is a distributed tracing system released as open source by Uber Technologies. It is used for monitoring and troubleshooting microservices-based distributed systems, including:

* Distributed context propagation
* Distributed transaction monitoring
* Root cause analysis
* Service dependency analysis
* Performance / latency optimization

[Kiali](https://www.kiali.io) includes [Jaeger Tracing](https://www.jaegertracing.io) to provide distributed tracing out of the box.

#### 

By default Istio provides tracing information for each service.
But you can see that it is information per service but do not have links between all services.
The reason is that we need to propagate 

#### What are you hidding?

From the [Kiali Console]({{ KIALI_URL }}), click on the `Distributed Tracing` link in the left navigation and enter the following configuration:

 * Select a Namespace: `{{COOLSTORE_PROJECT}}`
 * Select a Service: `gateway`
 * Then click on the **magnifying glass** on the right

![Jaeger - Traces View]({% image_path jaeger-trace-delay-view.png %}){:width="700px"}

You see all the traces containing Gateway Service.
On the left hand side of each traces, you have information like the duration.
One call takes more than 400ms which you could judge as *normal* but ...

Let’s click on the trace title bar.

![Jaeger - Trace Detail View]({% image_path jaeger-trace-delay-detail-view.png %}){:width="700px"}

Interesting!! You can see than the major part of the time is consuming by the Catalog Service.
So let's have a look on its code. 
Go through the Catalog Service source code and find the following piece of code.

~~~java
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
~~~

And yes, this burns your eyes right? Basically nobody could understand what the developer attempted to achieve but we do not have the time for that.
This piece of code is a part of the `getAll` method which returns the list of all products from the database. 
As you are an expert of Java 8, you are going to simplify the code and increase performance.

Replace the `getAll` method as following:

~~~java
    public List<Product> getAll() {
        Spliterator<Product> products = repository.findAll().spliterator();
        return StreamSupport.stream(products, false).collect(Collectors.toList());
    }
~~~

Now let's push the new version of the source code.

~~~shell
$ oc start-build catalog-s2i --from-dir labs/catalog-spring-boot/ --follow
~~~

![Jaeger - Trace Detail View]({% image_path jaeger-trace-fixed-detail-view.png %}){:width="700px"}

Just wonderful! A masterpiece! You should be proud!!