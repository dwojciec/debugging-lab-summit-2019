![Summit Logo]({% image_path redhatsummit2019.jpg %}){:width="500px"}

## Friday, 4:49PM, it begins...

Here the context

## Getting Started

First of all, go to the [CodeReady Workspaces url]({{CODEREADY_WORKSPACES_URL}}) in order to configure your development workspace.

#### Registering to CodeReady Workspaces
First, you need to register as a user. Register and choose the same username and password as 
`{{OPENSHIFT_USER}}/{{OPENSHIFT_PASWORD}}`.

![CodeReady Workspaces - Register]({% image_path codeready-register.png %}){:width="500px"}

#### Creating a Workspace
Log into CodeReady Workspaces with your user. You can now create your workspace based on a stack. A 
stack is a template of workspace configuration. For example, it includes the programming language and tools needed
in your workspace. Stacks make it possible to recreate identical workspaces with all the tools and needed configuration
on-demand. 

For this lab, click on the **Java Cloud-Native** stack and then on the **Create** button. 

![CodeReady Workspaces - Workspace]({% image_path codeready-create-workspace.png %}){:width="1000px"}

Click on **OPEN** to open and to start the workspace.

![CodeReady Workspaces - Workspace]({% image_path codeready-start-workspace.png %}){:width="1000px"}

It takes a little while for the workspace to be ready. When it's ready, you will see a fully functional CodeReady Workspaces IDE running in your browser.

![CodeReady Workspaces - Workspace]({% image_path codeready-workspace.png %}){:width="1000px"}

#### Importing the lab project
Now you can import the project skeletons into your workspace.

In the project explorer pane, click on the **Import Project...** and enter the following:

  * Type: `ZIP`
  * URL: `{{LABS_DOWNLOAD_URL}}`
  * Name: `labs`
  * Check **Skip the root folder of the archive**

![CodeReady Workspaces - Import Project]({% image_path codeready-import.png %}){:width="500px"}

Click on **Import**. Make sure you choose the **Blank** project configuration since the zip file contains multiple 
project skeletons. Click on **Save**

![CodeReady Workspaces - Import Project]({% image_path codeready-import-save.png %}){:width="500px"}

#### Accessing to OpenShift with OpenShift CLI

In order to login, we will use the `oc` command and then specify the server that we
want to authenticate to.

Issue the following command in CodeReady Workspaces terminal:

~~~shell
$ oc login {{OPENSHIFT_CONSOLE_URL}}
~~~

You may see the following output:

~~~shell
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n):
~~~

Enter in `Y` to use a potentially insecure connection.  The reason you received
this message is because we are using a self-signed certificate for this
workshop, but we did not provide you with the CA certificate that was generated
by OpenShift. In a real-world scenario, either OpenShift's certificate would be
signed by a standard CA (eg: Thawte, Verisign, StartSSL, etc.) or signed by a
corporate-standard CA that you already have installed on your system.

Enter the username and password `{{OPENSHIFT_USER}}/{{OPENSHIFT_PASWORD}}` provided to you by the instructor

Congratulations, you are now authenticated to the OpenShift server.

[Projects]({{OPENSHIFT_DOCS_BASE}}/architecture/core_concepts/projects_and_users.html#projects) 
are a top level concept to help you organize your deployments. An
OpenShift project allows a community of users (or a user) to organize and manage
their content in isolation from other communities. Each project has its own
resources, policies (who can or cannot perform actions), and constraints (quotas
and limits on resources, etc). Projects act as a "wrapper" around all the
application services and endpoints you (or your teams) are using for your work.

For this lab, let's connect to the project that you will use in the following labs for 
deploying your applications. 

> Make sure to use your dedicated project {{COOLSTORE_PROJECT}} by running the following command `oc project {{COOLSTORE_PROJECT}}`

OpenShift ships with a web-based console that will allow users to
perform various tasks via a browser.  To get a feel for how the web console
works, open your browser and go to the [OpenShift Web Console]({{OPENSHIFT_CONSOLE_URL}}).

The first screen you will see is the authentication screen. Enter your username and password `{{OPENSHIFT_USER}}/{{OPENSHIFT_PASWORD}}` and 
then log in. After you have authenticated to the web console, you will be presented with a
list of projects that your user has permission to work with. 

Click on the **{{COOLSTORE_PROJECT}}** project to be taken to the project overview page
which will list all of the routes, services, deployments, and pods that you have
running as part of your project. There's nothing there now, but that's about to
change.

Now you are ready to tackle the problemS!