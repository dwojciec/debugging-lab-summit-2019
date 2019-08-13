# debugging-lab-summit-2019
# Red Hat Summit 2019: 
# Getting Hands On Debugging Microservices Applications on OpenShift


== Purpose

As microservices-based applications become more prevalent, both the number of
and complexity of their interactions increases. Up until now much of the burden
of managing these complex microservices interactions has been placed on the
application developer, with different or non-existent support for microservice
concepts depending on language and framework.

Deploying and managing microservices architecture is becoming easier with a 
container orchestration platform like Openshift. So now, critical questions
 is raising: How do you debug a distributed system and how do you ensure 
the good health of your application? Debugging microservices architecture is hard. 
The state of the application is spread through multiple microservices which makes 
more complicated to get a global overview for debugging purposes.

== Background
This hands-on workshop will introduce and help you to get familiar with the common 
tracing/debugging techniques using Jaeger/OpenTracing, Istio, Kiali and Squash in 
a microservice context

# Agenda
* Kiali / Istio
* Opentracing - Jaeger
* Realtime Debugging with Squash

~~~
--------------------
Setup complete. Open the Lab Instructions in your browser: http://debug-workshop-guides.aa.bb.cc.dd.xip.io
--------------------
~~~

This is the URL for your customized lab guide that you will use for the rest of
the lab. Please open that URL in your browser and continue from there.

# Using this lab content elsewhere
== Deploy On OpenShift

You can deploy the lab guides as a container image anywhere but most
conveniently, you can deploy it on OpenShift Online or other OpenShift flavours:

```
oc new-project guides
oc new-app osevg/workshopper --name=debug-workshop \
      -e CONTENT_URL_PREFIX=https://raw.githubusercontent.com/dwojciec/debugging-lab-summit-2019/master/instructions
      -e WORKSHOPS_URLS="https://raw.githubusercontent.com/dwojciec/debugging-lab-summit-2019/master/instructions/_rhsummit19.yml" \
      -e OPENSHIFT_CONSOLE_URL="http://127.0.0.1:8443" \
      -e APPS_SUFFIX="apps.127.0.0.1.nip.io" \
      -e DEBUG_LAB_HOSTNAME="127.0.0.1" \
      -e LABS_DOWNLOAD_URL="https://github.com/mcouliba/cloud-native-labs/archive/debugging.zip" \                            
      -e CODEREADY_WORKSPACES_URL="http://codeready-lab-infra.apps.127.0.0.1.nip.io " \                              
      -e OPENSHIFT_USER=userX 
      -e OPENSHIFT_PASWORD=openshift                              \
      -e KIALI_URL="https://kiali-infraX.apps.127.0.0.1.nip.io/"     \                             
      -e COOLSTORE_TEMPLATE="https://raw.githubusercontent.com/dwojciec/debugging-lab-summit-2019/master/openshift/coolstore.yml"

oc expose svc/debug-workshop
```

Replace `OPENSHIFT_MASTER` with the URL to the console of your working OpenShift
environment (e.g.  `http://127.0.0.1:8443`), `APPS_SUFFIX` with your default
routing suffix (e.g.  `apps.127.0.0.1.nip.io`), and `DEBUG_LAB_HOSTNAME` with
the public hostname of your machine. These variables are used to subsitute
values in the markdown content files.

The guides can then be accessed at `http://debug-workshop-guides.$APPS_SUFFIX`.

The lab content (`.md` files) will be pulled from the GitHub when users access the guides in
their browser.

Note that the workshop variables can be overriden via specifying environment
variables on the container itself e.g. the `JAVA_APP` env var in the above
command

== Test Locally with Docker

You can directly run Workshopper as a docker container which is specially helpful when writing the content.

~~~shell
git clone https://github.com/dwojciec/debugging-lab-summit-2019.git
cd debugging-lab-summit-2019.git

docker run -p 8080:8080 -v $(pwd):/app-data \
              -e CONTENT_URL_PREFIX="file:///app-data/instructions" \
              -e WORKSHOPS_URLS="file:///app-data/instructions/_rhsummit19.yml" \
              -e OPENSHIFT_CONSOLE_URL=https://127.0.0.1:8443/" \                    
-e APPS_SUFFIX=“apps.127.0.0.1.nip.io" \                          
-e DEBUG_LAB_HOSTNAME="MY_HOSTNAME" \
-e LABS_DOWNLOAD_URL="https://github.com/mcouliba/cloud-native-labs/archive/debugging.zip" \                            
-e CODEREADY_WORKSPACES_URL="http://codeready-lab-infra.apps.127.0.0.1.nip.io " \                              
-e OPENSHIFT_USER=userX -e OPENSHIFT_PASWORD=openshift                              \
-e KIALI_URL="https://kiali-infraX.apps.127.0.0.1.nip.io/"     \                             
-e COOLSTORE_TEMPLATE= https://raw.githubusercontent.com/dwojciec/debugging-lab-summit-2019/master/openshift/coolstore.yml"   \  
              quay.io/osevg/workshopper:latest
~~~

Replace `OPENSHIFT_MASTER` with the URL to the console of your working OpenShift
environment (e.g.  `http://127.0.0.1:8443`), `APPS_SUFFIX` with your default
routing suffix (e.g.  `apps.127.0.0.1.nip.io`), and `DEBUG_LAB_HOSTNAME` with
the public hostname of your machine. These variables are used to subsitute
values in the markdown content files.

Go to `http://localhost:8080` on your browser to see the rendered workshop
content. You can modify the lab instructions and refresh the page to see the
latest changes.

