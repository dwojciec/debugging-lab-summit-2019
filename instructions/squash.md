## "Mr Robot"  Please help me
After checking logs and traces we need the ability to do live debugging of my application, it's an essential piece in the development process. It's time to enter to the system running. How to penetrate to the secured kubernetes system, I need the POWER of Elliot Alderson. 

![MrRobot]({% image_path mrrobot.png %}){:width="500px"}

[Source: https://www.usanetwork.com/mrrobot/photos/eps22init1asec](https://www.usanetwork.com/mrrobot/photos/eps22init1asec)

## What is SQUASH ?

[Solo.io](https://solo.io/) created [SQUASH](https://github.com/solo-io/squash) for their use, to assist on the development of their own projects like [Gloo](https://www.solo.io/glooe), a Next Generation API Gateway, and [Supergloo](https://www.solo.io/copy-of-glooe), a service mesh orchestration platform. Squash can be your daily friend on this journey, and for two fundamental reasons: made for cloud-native workloads and enterprise security concerns.
[Squash](https://squash.solo.io) is a distributed multi-language debugger that allows s to step by step debug our application.
[Solo.io](https://solo.io/) mission is to build tools and help people to adopt Service Mesh.

### For java developers
Let’s take a look at the flow below, which is for debugging a Java application, for example:

![Java]({% image_path java_squash.png %}){:width="700px"}

Squash brings excellent value to Java developers in that it will automatically find the debug port that is specified when the JVM starts. After the port is located, it uses port forward and then relies on the IDE’s capability to leverage JDWP.


### For Go developers
For Go software engineers that run and develop for Kubernetes, it’s fair to say that it’s a must-have. There are a [few ways to debug a Go application in Kubernetes](https://kubernetes.io/blog/2018/05/01/developing-on-kubernetes/), but none is as smooth and considerate of enterprise scenarios as Squash.

![Go]({% image_path go_squash.png %}){:width="700px"}


## Debugging Applications

In this lab you will debug the CoolStore application using Squash debugging tool and look into line-by-line code execution as the code runs inside a container on OpenShift.



#### Investigate The Bug

CoolStore application seems to have a bug that causes the inventory status for one of the products not to be displayed in the web interface

![Inventory Status Bug]({% image_path debug-coolstore-bug.png %}){:width="800px"}

This is not an expected behavior!

The Gateway pod is composed of **vertx** container and **istio-proxy** container.

~~~shell
$ oc logs dc/gateway -c vertx| grep -i error

...
WARNING: Inventory error for 444436: status code 204
...
~~~

Oh! Something seems to be wrong with the response the API Gateway has received from the 
Inventory API for the product id `444436`. 

Look into the Inventory pod logs to investigate further and see if you can find more  
information about this bug:


~~~shell
$ oc logs dc/inventory -c thorntail-v2 | grep ERROR
~~~

There doesn't seem to be anything relevant to the `invalid response` error that the 
API Gateway received either! 

Invoke the Inventory API using `curl` for the suspect product id to see what actually 
happens when API Gateway makes this call:

> You can find out the Inventory route url using `oc get route inventory`. Replace 
> `{{INVENTORY_ROUTE_HOST}}` with the Inventory route url from your project.

~~~shell
$ curl --verbose http://{{INVENTORY_ROUTE_HOST}}/api/inventory/444436
~~~
or use **Command** names **curl inventory GET API**
![Icons]({% image_path curl-inventory-1.png %}){:width="700px"}

> You can use `curl -v` to see all the headers sent and received. You would received 
> a `HTTP/1.1 204 No Content` response for the above request.
example

~~~
 curl http://inventory-coolstoreXX.apps.nantes-YYYY.openshiftworkshop.com/api/inventory/444436 -v
* About to connect() to inventory-coolstoreXX.apps.nantes-YYYY.openshiftworkshop.com port 80 (#0)
*   Trying 52.58.29.86...
* Connected to inventory-coolstoreXX.apps.nantes-YYYY.openshiftworkshop.com (52.58.29.86) port 80 (#0)
> GET /api/inventory/444436 HTTP/1.1
> User-Agent: curl/7.29.0
> Host: inventory-coolstoreXX.apps.nantes-YYYY.openshiftworkshop.com
> Accept: */*
>
< HTTP/1.1 204 No Content
< date: Mon, 15 Apr 2019 09:40:58 GMT
< x-envoy-upstream-service-time: 7
< server: istio-envoy
< x-envoy-decorator-operation: inventory.coolstoreXX.svc.cluster.local:8080/*
< Set-Cookie: 99431852768157cf108d330fd0eca9b9=e9242f754299538f7921ed6aa430982c; path=/; HttpOnly
< Cache-control: private
<
* Connection #0 to host inventory-coolstoreXX.apps.nantes-YYYY.openshiftworkshop.com left intact

~~~


No response came back and that seems to be the reason the inventory status is not displayed 
on the web interface.

Let's debug the Inventory service to get to the bottom of this!

#### Debugging with Squashctl  

Squash brings the power of modern popular debuggers to developers of microservices apps that run on container orchestrator platforms. Choose which containers, pods, services or images you want to debug, and Squash will let you set breakpoints, step through your code while jumping between microservices, follow variable values on the fly, and change these values during run time. 

Using **squashctl** on Inventory by running the following inside the `/ 
directory in the CodeReady Workspaces **Terminal** window:

~~~shell
$ squashctl --version
squashctl version 0.5.8, created 2019-04-09.21:00:55
~~~

The Java image on OpenShift has built-in support for remote debugging and it can be enabled by setting the **JAVA_DEBUG=true** environment variables on the deployment config for the pod that you want to remotely debug.

~~~shell
$ cd /projects/labs/inventory-thorntail
$ oc set env dc/inventory JAVA_DEBUG=true
$ oc get pods -lapp=inventory,deploymentconfig=inventory
NAME                           READY     STATUS    RESTARTS   AGE
inventory-1-l22lz              2/2       Running   2          26m
~~~

Using *squashctl* on the **Terminal** window or you can use the **Command** **Debug Squash Inventory**

~~~shell
$ squashctl --machine --namespace coolstoreXX --debugger java --squash-namespace infraXX
? Select a pod inventory-3-lqvmt
{"PortForwardCmd":"kubectl port-forward inventory-3-lqvmt :5005 -n coolstoreXX"}

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