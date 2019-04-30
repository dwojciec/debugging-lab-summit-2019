## "Mr Robot", Please Help Me!

*20 MINUTES PRACTICE*

After checking logs and traces we need the ability to do live debugging of my application, it's an essential piece in the development process. It's time to enter to the system running. How to penetrate to the secured kubernetes system, I need the POWER of Elliot Alderson. 

![MrRobot]({% image_path mrrobot.png %}){:width="500px"}

[Source: https://www.usanetwork.com/mrrobot/photos/eps22init1asec](https://www.usanetwork.com/mrrobot/photos/eps22init1asec)

#### What is Kibana?

![Kibana]({% image_path Kibana-Logo-Color-H.png %}){:width="200px"}

OpenShift provides a logging solution based on ElasticSearch, Fluentd, and [Kibana](https://en.wikipedia.org/wiki/Kibana) :

*  **Fluentd** which serves as both the collector and the normalizer, 
*  **Elasticsearch** serves as the warehouse, and 
*  **Kibana** is the visualizer (web UI). **Kibana** is a Node.js application. It works very well with Elasticsearch and is tightly coupled to it. 

The logging system can provide two views: 

* **Project logs** - access controlled view to specific project logs for running containers and project resources. Our case in this Workshop. 
* **Ops view** - aggregate logging of all projects across the cluster, in addition to platform logs (nodes, docker, and masters, for example). 

[Additionnal information](https://docs.openshift.com/container-platform/3.11/install_config/aggregate_logging.html#aggregate-logging-kibana)

Logs management plays a vital role when we encounter an error in the application. If we do not manage the logs, it will be difficult in any application, especially in microservices architecture to find the error and fix it. For our application with lots of microservices we have to identify interesting traces and Kibana is offering an nice User Interface with search field to explore and to analyze logs files easily. Whenever we get some error in the application(s), we can get the error details and analyze them in a simple way.


#### Investigate The Bug

CoolStore application seems to have a bug that causes the inventory status for one of the products not to be displayed in the web interface

![Inventory Status Bug]({% image_path debug-coolstore-bug.png %}){:width="800px"}

This is not an expected behavior!

Let's start our investigation from the application logs!
Log in to the [Kibana Console]({{ KIBANA_URL }}) as `{{ OPENSHIFT_USER }}/{{ OPENSHIFT_PASSWORD }}`

![Kibana - Console]({% image_path kibana-console.png %}){:width="600px"}

After you log in, enter the following configuration:

 * Selected Fields: **'kubernetes.pod_name'**, **'message'**
 * Search: **'message:(inventory AND error)'**

![Kibana - Search]({% image_path kibana-search.png %}){:width="200px"}

**Push the 'Enter' button**, you will get the following results:

![Kibana - Error Result]({% image_path kibana-error-result.png %}){:width="600px"}

Oh! Something seems to be wrong with the response the ***Gateway Service*** has received from the ***Inventory Service*** for the product id **'444436'**. 
But there doesn't seem to be anything relevant to the **invalid response** error at the ***Inventory Service*** level! 

**Go back to Distributed Tracing menu** from [Kiali Console]({{ KIALI_URL }}). **Select one of the Distributed Trace then on Search field enter the product id '444436'**. One span should be highlighted in *light yellow*.

![Jaeger - Trace Inventory ]({% image_path jaeger-trace-inventory.png %}){:width="600px"}

**Expand the 'inventory.{{ COOLSTORE_PROJECT }}' span** in order to get more detail.

![Jaeger - Trace Inventory ]({% image_path jaeger-trace-inventory-details.png %}){:width="800px"}

No response came back from ***Inventory Service*** for the product id **'444436'** and that seems to be the reason the inventory status is not displayed 
on the web interface.

Let's debug the ***Inventory Service*** to get to the bottom of this!

#### What is Squash ?

![Squash Logo]({% image_path squash-logo.png %}){:width="150px"}

[Solo.io](https://solo.io/) created [SQUASH](https://github.com/solo-io/squash) for their use, to assist on the development of their own projects like [Gloo](https://www.solo.io/glooe), a Next Generation API Gateway, and [Supergloo](https://www.solo.io/copy-of-glooe), a service mesh orchestration platform. Squash can be your daily friend on this journey, and for two fundamental reasons: made for cloud-native workloads and enterprise security concerns.
[Squash](https://squash.solo.io) is a distributed multi-language debugger that allows s to step by step debug our application.
[Solo.io](https://solo.io/) mission is to build tools and help people to adopt Service Mesh.

#### For Java developers
Let’s take a look at the flow below, which is for debugging a Java application, for example:

![Java]({% image_path java_squash.png %}){:width="700px"}

[source Medium - Squash, the definitive cloud-native debugging tool](https://medium.com/solo-io/squash-the-definitive-cloud-native-debugging-tool-9d10308fe1da)

Squash brings excellent value to Java developers in that it will automatically find the debug port that is specified when the JVM starts. After the port is located, it uses port forward and then relies on the IDE’s capability to leverage JDWP.

#### For Go developers
For Go software engineers that run and develop for Kubernetes, it’s fair to say that it’s a must-have. There are a [few ways to debug a Go application in Kubernetes](https://kubernetes.io/blog/2018/05/01/developing-on-kubernetes/), but none is as smooth and considerate of enterprise scenarios as Squash.

![Go]({% image_path go_squash.png %}){:width="700px"}

[source Medium - Squash, the definitive cloud-native debugging tool](https://medium.com/solo-io/squash-the-definitive-cloud-native-debugging-tool-9d10308fe1da)

#### Debugging with Squashctl

Squash brings the power of modern popular debuggers to developers of microservices apps that run on container orchestrator platforms. Choose which containers, pods, services or images you want to debug, and Squash will let you set breakpoints, step through your code while jumping between microservices, follow variable values on the fly, and change these values during run time. 

Using **squashctl** on Inventory by running the following inside the `/ 
directory in the CodeReady Workspaces **Terminal** window:

In CodeReady Workspaces, use the ***Commands Palette*** and **click on DEBUG > Squash Version**

~~~shell
$ squashctl --version
squashctl version 0.5.8, created 2019-04-09.21:00:55
~~~

The Java image on OpenShift has built-in support for remote debugging and it can be enabled by setting the **JAVA_DEBUG=true** environment variables on the deployment config for the pod that you want to remotely debug.

~~~shell
$ oc set env dc/inventory JAVA_DEBUG=true
$ oc get pods -lapp=inventory,deploymentconfig=inventory
NAME                           READY     STATUS    RESTARTS   AGE
inventory-1-l22lz              2/2       Running   2          26m
~~~

The status should be **Running** and there should be **2/2** pods in the **Ready** column. 

Using *squashctl* on the **Terminal** window or you can use the **Command** **Debug Squash Inventory**

~~~shell
$ squashctl --namespace coolstoreXX --debugger java-port --squash-namespace infraXX
Forwarding from 127.0.0.1:34930 -> 5005
Forwarding from [::1]:34930 -> 5005
Handling connection for 34930

~~~
or 
![Debug Inventory]({% image_path debug-inventory-0.png %}){:width="450px"}

![Debug Inventory]({% image_path debug-inventory.png %}){:width="700px"}

![Debug Inventory]({% image_path debug-inventory-1.png %}){:width="700px"}


You are all set now to start debugging. 

Forward the remote debugging port of the Inventory service locally by executing the following command generated by the command below into **"PortForwardCmd"**:

~~~shell
[user@workspace7gz4p7p2gxch7vps projects]$ kubectl port-forward inventory-3-nc62t :5005 -n coolstore22
Forwarding from 127.0.0.1:34930 -> 5005
Forwarding from [::1]:34930 -> 5005
Handling connection for 34930
~~~

Do not wait for the command to return! It keeps the forwarded 
port open so that you can start debugging remotely.

You can have a look in the infraXX project to see the pod created by squash.

![Squash pod]({% image_path debug-squash-pod.png %}){:width="700px"}

#### Remote Debug with CodeReady Workspaces

CodeReady Workspaces provides a convenient way to remotely connect to Java applications running 
inside containers and debug while following the code execution in the IDE.

From the **Run** menu, click on **Edit Debug Configurations...**.

![Remote Debug]({% image_path debug-che-debug-config-1.png %}){:width="600px"}

The window shows the debuggers available in CodeReady Workspaces. Click on the plus sign near the 
Java debugger.

![Remote Debug]({% image_path debug-che-debug-config-2.png %}){:width="700px"}

Configure the remote debugger and click on the **Save** button:

* Check **Connect to process on workspace machine**
* Port: `34930`

![Remote Debug]({% image_path debug-che-debug-config-3.png %}){:width="700px"}

You can now click on the **Debug** button to make CodeReady Workspaces connect to the 
Inventory service running on OpenShift.

You should see a confirmation that the remote debugger is successfully connected.

![Remote Debug]({% image_path debug-che-debug-config-4.png %}){:width="500px"}

Open `com.redhat.cloudnative.inventory.InventoryResource` and click once
on the editor sidebar on the line number of the first line of the `getAvailability()` 
method to add a breakpoint to that line. A start appears near the line to show a breakpoint 
is set.

![Add Breakpoint]({% image_path debug-che-breakpoint.png %}){:width="600px"}

Open a new **Terminal** window and use `curl` to invoke the Inventory API with the 
suspect product id in order to pause the code execution at the defined breakpoint.

Note that you can use the the following icons to switch between debug and terminal windows.


![Icons]({% image_path debug-che-window-guide.png %}){:width="700px"}

>  You can find out the Inventory route url using `oc get routes`. Replace 
> `{{ INVENTORY_ROUTE_HOST }}` with the Inventory route url from your project.

~~~
$ curl --verbose http://{{ INVENTORY_ROUTE_HOST }}/api/inventory/444436
~~~
or use **Command** names **curl inventory GET API**
![Icons]({% image_path curl-inventory-1.png %}){:width="700px"}


![Icons]({% image_path curl-inventory-2.png %}){:width="700px"} 

Switch back to the debug panel and notice that the code execution is paused at the 
breakpoint on `InventoryResource` class.

![Icons]({% image_path debug-che-breakpoint-stop.png %}){:width="900px"}

Click on the _Step Over_ icon to execute one line and retrieve the inventory object for the 
given product id from the database.

![Step Over]({% image_path debug-che-step-over.png %}){:width="340px"}

Click on the the plus icon in the **Variables** panel to add the `inventory` variable 
to the list of watch variables. This would allow you to see the value of `inventory` variable 
during execution.

![Watch Variables]({% image_path debug-che-variables.png %}){:width="500px"}

![Debug]({% image_path debug-che-breakpoint-values.png %}){:width="900px"}

Can you spot the bug now? 

Look at the **Variables** window. The retrieved inventory object is `null`!

The non-existing product id is not a problem on its own because it simply could mean 
this product is discontinued and removed from the Inventory database but it's not 
removed from the product catalog database yet. The bug is however caused because 
the code returns this `null` value instead of a sensible REST response. If the product 
id does not exist, a proper JSON response stating a zero inventory should be 
returned instead of `null`.

Click on the _Resume_ icon to continue the code execution and then on the stop icon to 
end the debug session.

#### Additionnal Squash command (normally not needed here)
To delete planks created in our namespace

~~~
squashctl utils delete-planks --squash-namespace infraXX
~~~
or using the **Command** **Delete Squash Attachments** 
![Debug]({% image_path delete-attachments.png %}){:width="450px"}

![Debug]({% image_path delete-attachments-1.png %}){:width="900px"}

#### Fix the Inventory Bug

Edit the `InventoryResource.java` and update the `getAvailability()` to make it look like the following 
code in order to return a zero inventory for products that don't exist in the inventory 
database:

~~~java
@GET
@Path("/api/inventory/{itemId}")
@Produces(MediaType.APPLICATION_JSON)
public Inventory getAvailability(@PathParam("itemId") String itemId) {
    Inventory inventory = em.find(Inventory.class, itemId);

    if (inventory == null) {
        inventory = new Inventory();
        inventory.setItemId(itemId);
        inventory.setQuantity(0);
    }

    return inventory;
}
~~~

Go back to the **Terminal** window. Press 
`Ctrl+C` to stop the debug and port-forward and then run the following commands 
to rebuild the container with the code corrected.

~~~shell
$ oc start-build inventory-s2i --from-dir /projects/labs/inventory-thorntail/ --follow 
~~~
or use **Command** names **Build inventory Service**
![Build Inventory]({% image_path build-inventory-service.png %}){:width="700px"}


When the container is rebuilt and deployed, point your browser at the Web route and verify 
that the inventory status is visible for all products. The suspect product should show 
the inventory status as _Not in Stock_.

![Inventory Status Bug Fixed]({% image_path debug-coolstore-bug-fixed.png %}){:width="800px"}

Well done and congratulations for completing all the labs.

#### Optional part - To go futher.

To see the added value of squash we decided to deploy a new version of the catalog but here written in [GO](https://golang.org/) language.
In this lab you will see how you can use Site Mesh to do some A/B testing using and route traffic between 2 versions of the Catalog service.
[A/B testing](https://en.wikipedia.org/wiki/A/B_testing) allows running multiple versions of a functionality in parallel; and using analytics of the user behavior it is possible to determine which version is the best. It is also possible to launch the new features only for a small set of users, to prepare the general avalability of a new feature.

#### Deploying the new Catalog service

A new ***Catalog Service v2*** has been created, this service is developed in [Golang](https://golang.org/) and available in the following directory:

/projects/labs/catalog-go

This service use the same business logic except that all product descriptions are returned in uppercase.

Let's deploy the service directly from the catalog-go directory using the `oc new-app` command.

In the terminal window type the following command:

~~~shell
$ oc new-app /projects/labs/ --strategy=docker --context-dir=catalog-go --name=catalog-v2 --labels app=catalog,group=com.redhat.cloudnative,provider=fabric8,version=2.0 
$ oc start-build catalog-v2 --from-dir /projects/labs/catalog-go/ --follow 
 
~~~
or using the **Command** **Options: Catalog in Go**
![Catalog-in-Go]({% image_path Catalog-in-Go.png %}){:width="450px"}

> **Note**: To simplify the lab, we use the same labels for ***catalog*** and ***catalog-v2***, since they are used for the service routing.

Service Mesh will be used to route the traffic between the catalog service v1 and v2, so you have to add the Istio sidecar to the ***Catalog Service v2*** using the following command:

~~~shell
$ oc patch dc/catalog-v2 --patch \
  '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}'
~~~

Note: the command above **Options: Catalog in Go** is executing the "patching" of the deployment config of catalog-v2. 

To confirm that the application is successfully deployed, run this command:

~~~shell
$ oc get pods -lapp=catalog,deploymentconfig=catalog-v2
NAME                 READY     STATUS    RESTARTS   AGE
catalog-v2-2-7zsxb   2/2       Running   0          1m
~~~

The status should be **Running** and there should be **2/2** pods in the **Ready** column.
Wait few seconds that the application restarts.


#### Enabling A/B Testing

[A/B Testing](https://en.wikipedia.org/wiki/A/B_testing) allows to run in parallel two versions of an application with one single variant (usually visual) and to collect metrics in order to determine the variant with the best effect of the user behavior.

The implementation of such procedure is one are the advantages coming with OpenShift Service Mesh.


Let's now create the ***Destination Rule*** resource.

* A ***Destination Rule*** defines policies that apply to traffic intended for a service after routing has occurred. These rules specify configuration for load balancing, connection pool size from the sidecar, and outlier detection settings to detect and evict unhealthy hosts from the load balancing pool.

In the Terminal window, issue the following command:

~~~shell
$ cat << EOF | oc create -f -
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: catalog
spec:
  host: catalog
  subsets:
  - labels:
      version: "v1.0"
    name: "version-springboot"
  - labels:
      version: "2.0"
    name: "version-go"
EOF
~~~

Now you have created a ***Destination Rule*** for ***Catalog Service*** and ***Catalog Service v2***.

From Kiali UI using Services -> catalog -> Destination Rules
![Destination-Rule]({% image_path Destination-Rule.png %}){:width="900px"}

The last step is to define the rules to distribute the traffic between the services. 

* A **VirtualService** defines a set of traffic routing rules to apply when a host is addressed. Each routing rule defines matching criteria for traffic of a specific protocol. If the traffic is matched, then it is sent to a named destination service (or subset/version of it) defined in the registry.

In the Terminal window, issue the following command:

~~~shell
$ cat << EOF | oc create -f -
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: catalog
spec:
  hosts:
    - catalog
  http:
  - route:
    - destination:
        host: catalog
        subset: "version-springboot"
      weight: 50
    - destination:
        host: catalog
        subset: "version-go"
      weight: 50
EOF
~~~

Doing so, you route **50%** of the **HTTP traffic** to pods of the ***Catalog Service*** *(subset "version-springboot" ie label "version: v1.0")* and the **50%** remaining to pods of the ***Catalog Service v2*** *(subset "version-go" ie label "version: 2.0")*.

From Kiali UI using Services -> catalog -> Virtual Services
![Virtual-services]({% image_path Virtual-services.png %}){:width="900px"}


#### Generate HTTP traffic.

Let's now see the A/B testing with Site Mesh in action.
First, we need to generate HTTP traffic by sending several requests to the ***Gateway Service*** from the ***Istio Gateway***

In CodeReady Workspaces, click on ***Commands Palette*** and click on **RUN > testGatewayService**
![Commands Palette - RunGatewayService]({% image_path  codeready-command-run-gateway-service.png %}){:width="600px"}

You likely see *'Gateway => Catalog Spring Boot (v1)'* or *'Gateway => Catalog GoLang (v2)'*

![Terminal - RunGatewayService]({% image_path  codeready-run-gateway-50-50.png %}){:width="900px"}

> You can also go to the Web interface and refresh the page to see that product descriptions is sometimes in uppercase (v2) or not (v1).

Go to Kiali to see the traffic distribution between Catalog v1 and v2.

From the [Kiali Console]({{ KIALI_URL }}) *(please make sure to replace **infrax** with your dedicated project)*, `click on the 'Graph' link` in the left navigation and enter the following configuration:

 * Namespace: **{{ COOLSTORE_PROJECT }}**
 * Display: **check 'Traffic Animation'**
 * Edge Label: **Requests percent of total**
 * Fetching: **Last 5 min**

![Kiali- Graph]({% image_path kiali-abtesting-50-50.png %}){:width="700px"}

You can see that the traffic between the two version of the ***Catalog*** is shared equitably (at least very very close). 

After one week trial, you have collected enough information to confirm that product descriptions in uppercase do increate sales rates. So you will route all the traffic to ***Catalog Service v2***. Go back to the Terminal and run the following command:

~~~shell
$ cat << EOF | oc replace -f -
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: catalog
spec:
  hosts:
    - catalog
  http:
  - route:
    - destination:
        host: catalog
        subset: "version-springboot"
      weight: 0
    - destination:
        host: catalog
        subset: "version-go"
      weight: 100
EOF
~~~

Now, you likely see only *'Gateway => Catalog GoLang (v2)'* in the *'testGatewayService'* terminal.

![Terminal - RunGatewayService]({% image_path  codeready-run-gateway-100.png %}){:width="900px"}

And from [Kiali Console]({{ KIALI_URL }}) *(please make sure to replace **infrax** with your dedicated project)*, you can visualize that **100%** of the traffic is switching gradually to ***Catalog Service v2***.

![Kiali- Graph]({% image_path kiali-abtesting-100.png %}){:width="700px"}

#### How to debug GO code with gdb using squash


~~~shell
$ oc get pod | grep v2
$ squashctl  --namespace coolstore22 --debugger gdb --squash-namespace infra22
~~~

![debug-gdb-1]({% image_path debug-gdb-1.png %}){:width="900px"}

![debug-gdb-2]({% image_path debug-gdb-2.png %}){:width="900px"}

![debug-gdb-3]({% image_path debug-gdb-3.png %}){:width="900px"}
Configure the remote debugger and click on the **Save** button:

* Check **Connect to process on workspace machine**
* Port: `35398`
Now you can [debug GO using gdb](https://golang.org/doc/gdb)

![debug-gdb-4]({% image_path debug-gdb-4.png %}){:width="900px"}

That's all for this lab! You are ready to move on to the next lab.