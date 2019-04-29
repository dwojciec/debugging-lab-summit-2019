## "Mr Robot", Please Help Me!

*20 MINUTES PRACTICE*

After checking logs and traces we need the ability to do live debugging of my application, it's an essential piece in the development process. It's time to enter to the system running. How to penetrate to the secured kubernetes system, I need the POWER of Elliot Alderson. 

![MrRobot]({% image_path mrrobot.png %}){:width="500px"}

[Source: https://www.usanetwork.com/mrrobot/photos/eps22init1asec](https://www.usanetwork.com/mrrobot/photos/eps22init1asec)

#### What is Kibana?

![Kibana]({% image_path Kibana-Logo-Color-H.png %}){:width="200px"}
OpenShift provides a logging solution based on ElasticSearch, Fluentd, and Kibana.: 
*  **Fluentd** which serves as both the collector and the normalizer, 
*  **Elasticsearch** serves as the warehouse, and 
*  **Kibana** is the visualizer (web UI). **Kibana** is a Node.js application.  It works very well with Elasticsearch and is tightly coupled to it. 

The logging system can provide two views: 
* Project logs - access controlled view to specific project logs for running containers and project resources. Our case in this Workshop. 
* Ops view - aggregate logging of all projects across the cluster, in addition to platform logs (nodes, docker, and masters, for example). 

[Additionnal information](https://docs.openshift.com/container-platform/3.11/install_config/aggregate_logging.html#aggregate-logging-kibana)



<TO BE COMPLETED - what it is and the benefits for a Microservice Architecture>

#### Investigate The Bug

CoolStore application seems to have a bug that causes the inventory status for one of the products not to be displayed in the web interface

![Inventory Status Bug]({% image_path debug-coolstore-bug.png %}){:width="800px"}

This is not an expected behavior!

Let's start our investigation from the application logs!
Log in to the [Kibana Console]({{ KIBANA_URL }}) as `{{OPENSHIFT_USER}}`/`{{OPENSHIFT_PASWORD}}`

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

**Expand the 'inventory.{{COOLSTORE_PROJECT}}' span** in order to get more detail.

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
> `{{INVENTORY_ROUTE_HOST}}` with the Inventory route url from your project.

~~~
$ curl --verbose http://{{INVENTORY_ROUTE_HOST}}/api/inventory/444436
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

#### Additionnal Squash command 
To delete all planks

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