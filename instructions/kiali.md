## "Bird Box"... Not Today!!

Your *Developer Workspaces* is now up and running but you still do not know anything about the *Mysterious* Application.
Going all over this application and debugging it completely blindfolded is time consuming and a crazy bid as Malorie does in *Bird Box*.

![BirdBox]({% image_path birdbox.png %}){:width="300px"}

Red Hat OpenShift Container Platform provides services to get observability of applications and to understand how different components are interacting with each other.

#### Login to OpenShift

First, you need to access to the OpenShift cluster from [CodeReady Workspaces url]({{CODEREADY_WORKSPACES_URL}}).
In CodeReady Workspaces, click on `Commands Palette` and click on `OPENSHIFT > oc login`

![oc login]({% image_path codeready-command-oc-login.png %}){:width="300px"}

> **Command Palette Info**
>
> The command `oc login {{OPENSHIFT_CONSOLE_URL}}` is issued using the credentials `{{OPENSHIFT_USER}}/{{OPENSHIFT_PASWORD}}`

You should get an output in the `oc login` terminal as following:

~~~shell
Login successful.
 
You have access to the following projects and can switch between them with 'oc project <projectname>':
 
  * {{COOLSTORE_PROJECT}}
    {{INFRA_PROJECT}}
 
Using project "{{COOLSTORE_PROJECT}}".
Already on project "{{COOLSTORE_PROJECT}}" on server "{{OPENSHIFT_CONSOLE_URL}}:443".
-----------
Successful Connected to OpenShift as {{OPENSHIFT_USER}}
-----------
~~~

#### Build and Deploy the Mysterious Application

Once logged, you can build and deploy the application to debug  on OpenShift.
In CodeReady Workspaces, click on `Commands Palette` and click on `BUILD > Build Mysterious Application`

![oc login]({% image_path codeready-command-build-app.png%}){:width="300px"}

> **Command Palette Info**
>
> First, the `oc create` command creates a list of objects defining the application. 
> Then, the `oc start-build` commands build container images of all microservices from the local source code 
> and deploy them on OpenShift.
>
> This operation could take 5-10 minutes. Please, be patient :-)

You can observe the build and deployment progress from the [OpenShift Web Console]({{OPENSHIFT_CONSOLE_URL}}).

The first screen you will see is the authentication screen. Enter your username and password `{{OPENSHIFT_USER}}/{{OPENSHIFT_PASWORD}}` and 
then log in. After you have authenticated to the web console, you will be presented with a
list of projects that your user has permission to work with. 

Click on the `{{COOLSTORE_PROJECT}}` project to be taken to the project overview page
which will list all of the routes, services, deployments, and pods that you have
running as part of your project.

Once successfully built, deployed and runned on Openshift, the **6 pods** of your application should be *all in Dark Blue* as following:

![oc login]({% image_path openshift-console-application.png%}){:width="500px"}

Point your browser at the Web UI route url. You should be able to see the CoolStore with all 
products and their inventory status.

![CoolStore Shop]({% image_path coolstore-web.png %}){:width="840px"}

> In order to generate traffic, please refresh this page several times.

The *mysterious* application is now up and running. You can see that it is composed of several components but so far, you have no clue about how they are interacting together.

What's more, everything **seems** doing great but... 

#### What is Kiali?

A Microservice Architecture breaks up the monolith into many smaller pieces that are composed together. Patterns to secure the communication between services like fault tolerance (via timeout, retry, circuit breaking, etc.) have come up as well as distributed tracing to be able to see where calls are going.

A service mesh can now provide these services on a platform level and frees the application writers from those tasks. Routing decisions are done at the mesh level.

[Kiali](https://www.kiali.io) works with Istio, in OpenShift or Kubernetes, to visualize the service mesh topology, to provide visibility into features like circuit breakers, request rates and more. It offers insights about the mesh components at different levels, from abstract Applications to Services and Workloads.

#### "Kiali, please tell me, how is the application working?"

Kiali provides an interactive graph view of your namespace in real time, being able to display the interactions at several levels (applications, versions, workloads), with contextual information and charts on the selected graph node or edge.

First, you need to access to Kiali. 
Launch a browser and navigate to [Kiali Console]({{ KIALI_URL }}) ({{ KIALI_URL }}). 
You should see the Kiali console login screen.

![Kiali - Log In]({% image_path kiali-login.png %}){:width="500px"}

Log in to the Kiali console as `{{OPENSHIFT_USER}}`/`{{OPENSHIFT_PASWORD}}`

After you log in, click on the `Graph` link in the left navigation and enter the following configuration:

 * Namespace: `{{COOLSTORE_PROJECT}}`
 * Display: check `Traffic Animation`
 * Fetching: `Last min`

![Kiali - Graph]({% image_path kiali-graph.png %}){:width="700px"}

 This page shows a graph with all the microservices, connected by the requests going through them. On this page, you can see how the services interact with each other.

Even if the application *seemed* working fine, you can see from [Kiali Console]({{ KIALI_URL }}) ({{ KIALI_URL }}), there is a problem in the Gateway Service which sends a 4xx http error.

![Kiali - 4xx]({% image_path kiali-4xx.png %}){:width="300px"}

Open the Javascript Console from your browser, and you will find a 404 error when calling the `gateway/api/cart` API.

![Gateway Error]({% image_path gateway-cart-missing.png %}){:width="700px"}

Indeed, when you check the APIs exposed by the gateway, you cannot find any `/api/cart/id-*` one.

Let's fix it!!

#### Build and deploy the Quarkus microservice, the Cart Service

[Quarkus](https://quarkus.io/) is a Kubernetes Native Java stack tailored for GraalVM & OpenJDK HotSpot, crafted from the best of breed Java libraries and standards.

* Architectured for running in serverless and container environments like Knative and OpenShift. 
* Designed around a **container first philosophy**, what this means in real terms is that Quarkus is optimised for low memory usage and fast startup times.

We already compiled the Cart Service application to a native executable called `cart-1.0-SNAPSHOT-runner`. You can find in the `cart-quarkus` project under the `src/target`folder. It improves the startup time of the `Cart Service`, and produces a minimal disk footprint. The executable would have everything to run the application including the "JVM" and the application.

In this chapter, you will focus on creating a Docker image using the produced native executable.

![Quarkus - Container]({% image_path containerization-process.png %}){:width="700px"}

> If you want, take a moment to examine the source code of the Cart Service implemented with [Quarkus](https://quarkus.io/).
> You can find it under the package `com.redhat.cloudnative` in the `src/main/java` directory of the `cart-quarkus` project.

In the *Terminal* window, execute the following commands to leverage the build mechanism of OpenShift and deploy the service:

~~~bash
# To build the image on OpenShift
$ oc new-build --binary --name=cart -lapp=cart,version=v1.0
$ oc patch bc/cart -p '{"spec":{"strategy":{"dockerStrategy":{"dockerfilePath":"src/main/docker/Dockerfile"}}}}'
$ oc start-build cart --from-dir /projects/labs/cart-quarkus --follow

# To instantiate the image
$ oc new-app --image-stream=cart:latest -lapp=cart,version=v1.0

# To deploy an Istio SideCar and configure Catalog Service Deployment
$ oc rollout pause dc/cart
$ oc patch dc/cart --patch '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}'
$ oc set env dc/cart CATALOG_ENDPOINT=http://catalog:8080
$ oc rollout resume dc/cart

# To create the route
$ oc expose svc cart
~~~

![Openshift Console Cart]({% image_path console-cart.png %}){:width="500px"}

**YOU HAVE TO SEE THAT!** 
Have a look to the log of the `Cart Service` pod by cliking in the dark blue circle and then **just admire its amazing FAST BOOT TIME!**

~~~bash
2019-04-01 20:13:35,623 INFO  [io.quarkus] (main) Quarkus 0.11.0 started in 0.009s. Listening on: http://0.0.0.0:8080 
2019-04-01 20:13:35,623 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy, resteasy-jsonb, smallrye-rest-client]
2019-04-01 20:17:08,790 INFO  [com.red.clo.ser.ShoppingCartService] (XNIO-1 task-1) Using local cache for cart data
~~~ 

**AND YES, IT'S A JAVA APPLICATION!**

You can ensure the proper functioning of the `Cart Service` by accessing to {{CART_ROUTE_HOST}} and click on `Test it`.

![Cart Service]({% image_path cart-service.png %}){:width="500px"}

#### Update Gateway Service

Previously, we deployed the `Cart Service`. Now, you have to take it in account in the `Gateway Service`.
Edit the `src/main/java/com/redhat/cloudnative/gateway/GatewayVerticle.java` class as following:

First, add the WebClient attribute `cart` in the class `GatewayVerticle`

~~~java
    private WebClient catalog;
    private WebClient inventory;
    // Cart Attribute
    private WebClient cart; 
~~~

Then, define the route `/api/cart/:cardId` in the `start()` method

~~~java
        router.get("/health").handler(ctx -> ctx.response().end(new JsonObject().put("status", "UP").toString()));
        router.get("/api/products").handler(this::products);
        // Cart Route
        router.get("/api/cart/:cardId").handler(this::getCartHandler);
~~~

Next, replace the `ServiceDiscovery.create()` call as following

~~~java
        ServiceDiscovery.create(vertx, discovery -> {
            // Catalog lookup
            Single<WebClient> catalogDiscoveryRequest = HttpEndpoint.rxGetWebClient(discovery,
                    rec -> rec.getName().equals("catalog"))
                    .onErrorReturn(t -> WebClient.create(vertx, new WebClientOptions()
                            .setDefaultHost(System.getProperty("catalog.api.host", "localhost"))
                            .setDefaultPort(Integer.getInteger("catalog.api.port", 9000))));

            // Inventory lookup
            Single<WebClient> inventoryDiscoveryRequest = HttpEndpoint.rxGetWebClient(discovery,
                    rec -> rec.getName().equals("inventory"))
                    .onErrorReturn(t -> WebClient.create(vertx, new WebClientOptions()
                            .setDefaultHost(System.getProperty("inventory.api.host", "localhost"))
                            .setDefaultPort(Integer.getInteger("inventory.api.port", 9001))));

            // Cart lookup
            Single<WebClient> cartDiscoveryRequest = HttpEndpoint.rxGetWebClient(discovery,
                    rec -> rec.getName().equals("cart"))
                    .onErrorReturn(t -> WebClient.create(vertx, new WebClientOptions()
                            .setDefaultHost(System.getProperty("inventory.api.host", "localhost"))
                            .setDefaultPort(Integer.getInteger("inventory.api.port", 9002))));

            // Zip all 3 requests
            Single.zip(catalogDiscoveryRequest, inventoryDiscoveryRequest, cartDiscoveryRequest, 
                (cg, i, ct) -> {
                    // When everything is done
                    catalog = cg;
                    inventory = i;
                    cart = ct;
                    return vertx.createHttpServer()
                        .requestHandler(router::accept)
                        .listen(Integer.getInteger("http.port", 8080));
                }).subscribe();
        });
~~~

Finally, add the `getCartHandler` method in the `GatewayVerticle` class.

~~~java
    private void getCartHandler(RoutingContext rc) {
        String cardId = rc.request().getParam("cardId");
        
        // Retrieve cart
        cart
        .get("/api/cart/" + cardId)
        .as(BodyCodec.jsonObject())
        .rxSend()
        .subscribe(
            resp -> {
                if (resp.statusCode() != 200) {
                    new RuntimeException("Invalid response from the cart: " + resp.statusCode());
                }
                rc.response().end(Json.encodePrettily(resp.body()));
            },
            error -> rc.response().end(new JsonObject().put("error", error.getMessage()).toString())
        );
    }
~~~

Check that your source code compiles then use the OpenShift CLI command to start a new build and deployment for the update `Gateway Service`:

~~~shell
$ mvn clean package -f /projects/labs/gateway-vertx/
$ oc start-build gateway-s2i --from-dir /projects/labs/gateway-vertx/ --follow
~~~

Once deployed, check your javascript console that the *404 error* has disappeared.
In Kiali Graph, the Gateway Service is now green and you can see the new `Cart Service` is now present! 

![Gateway Fixed]({% image_path gateway-cart-fixed.png %}){:width="700px"}

**CONGRATULATIONS!!!** You survive and you put off the blindfold on your own. But it is not THE END...

Now,let's go deeper!!