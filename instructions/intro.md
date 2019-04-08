![Summit Logo]({% image_path redhatsummit2019.jpg %}){:width="200px"}

## May 7th, 2019 - 3:45PM - somewhere in Boston, it begins...

After a long and frustrated morning spending hours to try to solve an issue that a colleague, passing by chance with his/her cup of coffee, fixed in literally 5 seconds, you are finally ready to start your normal working day when ...

Suddently, your manager comes out of his/her office and walks to you with the very known face that saying *"You are in trouble!"*.
Indeed, the new release of a *Mysterious* Application is planned for tonight (everything is always urgent) and it seems to contain some bugs.

And of course, the developer who has worked on this last release is now drinking some mojitos on the paradise island and you are the only one available who can save the situation. Your manager points the source code [*Mysterious* Application](https://github.com/mcouliba/cloud-native-labs/tree/debugging) out, gives you a gently tap on the shoulder and says "Bon Courage!".

At that moment in time, you feel like you are in an island but not the same one than your colleague.

![Cast Away]({% image_path castaway.jpg %}){:width="300px"}

Now you have to:

* Identify the right tools to build and run the *Mysterious* Application
* Install them on your environment
* Build your application which could be composed by several heterogeneous languages
* Test, deploy, run it
* ...

Piouf!! A lot of things to do in a short period of time...

*"No Worry Be Happy"*, Red Hat OpenShift Container Platform, Inc is here to help!!

## What is CodeReady Workspaces?

![CodeReady]({% image_path codeready.png %}){:width="400px"}

[Red Hat CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview/) is a collaborative Kubernetes-native development platform that delivers OpenShift workspaces and an IDE for rapid cloud application development.

Built on the open Eclipse Che project, [Red Hat CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview/) provides developer workspaces on OpenShift with all the tools and the dependencies that are needed to code, build, test, run, and debug containerized applications. The entire product runs in the cloud and eliminates the need to install anything on a local machine.

* It offers fast onboarding capabilities for teams with powerful collaboration, workspace automation, and management at scale
* It removes inconsistencies and the “works on my machine” syndrome
* It protects source code from the hard-to-secure developer and personal laptops

## Getting your Developer Workspace with a single click

[Red Hat CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview/) will provide you an out-of-box *Developer Workspace* with all the tools and the dependencies we need to do the job.
And in only one single click!

First, go to [*Mysterious* Application](https://github.com/mcouliba/cloud-native-labs/tree/debugging). The `README.md` file contains a link called *"Developer Workspace"*. 

![Developer Workspace - Link]({% image_path developer-workspace-link.png %}){:width="800px"}

Click on it, login as `{{OPENSHIFT_USER}}/{{OPENSHIFT_PASWORD}}` and let's the magic happens...

![Developer Workspace - Build]({% image_path developer-workspace-build.png %}){:width="600px"}

> Red Hat CodeReady Workspaces uses a [Factory](https://developers.redhat.com/crw-fmi#share_workspaces_with_factories) to automate the provisioning of a specific workspace by using the `.factory.json` file in the GitHub repository.
>
> Providing a .factory.json file inside the repository signals to CodeReady Workspaces URL factory to configure the project and runtime according to this configuration file.

Once completed, you will have a fully functional CodeReady Workspaces IDE running in your browser within the source code already imported.

![CodeReady Workspaces - Workspace]({% image_path codeready-workspace.png %}){:width="800px"}

**CONGRATULATIONS!!!** You find a friend *Wilson* aka OpenShift.

![Cast Away - Wilson]({% image_path castaway-wilson.jpg %}){:width="300px"}

You are now ready to tackle all the problem*S*!