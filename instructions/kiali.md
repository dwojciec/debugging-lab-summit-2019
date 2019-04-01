## "Bird Box"... Not Today!!

Context BlaBlaBla

![BirdBox]({% image_path birdbox.png %}){:width="300px"}

#### Running the application on OpenShift

~~~shell
$ oc create -f {{COOLSTORE_TEMPLATE}}
$ oc start-build catalog-s2i --from-dir labs/catalog-spring-boot/
$ oc start-build inventory-s2i --from-dir labs/inventory-thorntail/
$ oc start-build gateway-s2i --from-dir labs/gateway-vertx/
$ oc start-build web --from-dir labs/web-nodejs/
~~~

From the [OpenShift Web Console]({{OPENSHIFT_CONSOLE_URL}}), you can see that you have installed a lot of components
but you still do not understand how they are interacting together. 


Point your browser at the Web UI route url. You should be able to see the CoolStore with all 
products and their inventory status.

![CoolStore Shop]({% image_path coolstore-web.png %}){:width="840px"}

> In order to generate traffic, please refresh this page several times.

Everything *seems* doing great but...

#### What is Kiali?

A Microservice Architecture breaks up the monolith into many smaller pieces that are composed together. Patterns to secure the communication between services like fault tolerance (via timeout, retry, circuit breaking, etc.) have come up as well as distributed tracing to be able to see where calls are going.

A service mesh can now provide these services on a platform level and frees the application writers from those tasks. Routing decisions are done at the mesh level.

[Kiali](https://www.kiali.io) works with Istio, in OpenShift or Kubernetes, to visualize the service mesh topology, to provide visibility into features like circuit breakers, request rates and more. It offers insights about the mesh components at different levels, from abstract Applications to Services and Workloads.

#### "Kiali, tell me, how does my application work?"

Kiali provides an interactive graph view of your namespace in real time, being able to display the interactions at several levels (applications, versions, workloads), with contextual information and charts on the selected graph node or edge.

First, you need to access to Kiali. 
Launch a browser and navigate to [Kiali Console]({{ KIALI_URL }}) ({{ KIALI_URL }}). 
You should see the Kiali console login screen.

![Kiali- Log In]({% image_path kiali-login.png %}){:width="500px"}

Log in to the Kiali console as `{{OPENSHIFT_USER}}`/`{{OPENSHIFT_PASWORD}}`

After you log in, click on the `Graph` link in the left navigation and enter the following configuration:

 * Namespace: `{{COOLSTORE_PROJECT}}`
 * Display: check `Traffic Animation`
 * Fetching: `Last min`

![Kiali- Graph]({% image_path kiali-graph.png %}){:width="700px"}

 This page shows a graph with all the microservices, connected by the requests going through them. On this page, you can see how the services interact with each other.

Even if the application *seemed* working fine, you can see there is a problem in the Gateway Service, inbound requests.
Please open the Javascript Console from your browser, and you will find a 404 error when calling the `gateway/api/cart` API.

![Gateway Error]({% image_path gateway-cart-missing.png %}){:width="700px"}

Indeed, when you check the APIs exposed by the gateway, you cannot find any `/api/cart` one.

Let's fix it!!

#### Adding the Cart Service

~~~bash
# To build the image on OpenShift
$ oc new-build --binary --name=cart -l app=cart
$ oc patch bc/cart -p '{"spec":{"strategy":{"dockerStrategy":{"dockerfilePath":"src/main/docker/Dockerfile"}}}}'
$ oc start-build cart --from-dir labs/cart-quarkus --follow

# To instantiate the image
$ oc new-app --image-stream=cart:latest -l app=cart,version=1.0

# To configure Catalog Service Deployment
$ oc rollout pause dc/cart
$ oc patch dc/cart --patch '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}'
$ oc set env dc/cart CATALOG_ENDPOINT=http://catalog:8080
$ oc rollout resume dc/cart

# To create the route
$ oc expose svc cart
~~~

![Openshift Console Cart]({% image_path console-cart.png %}){:width="700px"}

#### Updating Gateway Service

~~~java
router.get("/api/cart/:cardId").handler(this::getCartHandler);

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

private void getCartHandler(RoutingContext rc) {
        String cardId = rc.request().getParam("cardId");
        
        // Retrieve catalog
        TracingInterceptor.propagate(cart, rc)
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
    }
~~~

Push the updated version

~~~shell
$ oc start-build gateway-s2i --from-dir labs/gateway-vertx/ --follow
~~~

You see the 404 error if fixed (Check the Javascript) and the Gateway Service in Kiali Graph is now green!!

![Gateway Fixed]({% image_path gateway-cart-fixed.png %}){:width="700px"}

Now let's go deeper!!