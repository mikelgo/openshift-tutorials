# Deploying Applications From Images

### Goal
Learn how to deploy an application on OpenShift with the web console and with the `oc` command line tool.

### Concepts

*   Deploying existing container images on an OpenShift cluster
*   OpenShift Web Console’s Topology view
*   OpenShift Projects and Applications
*   OpenShift `oc` tool’s `new-app` subcommand


### Use case

You can deploy a container image on an OpenShift cluster to make the application easier to manage, scale, connect and monitor.


# Topic 1 - Creating an Initial Project

Before we get started, you need to login and create a project in OpenShift to work in.

To login to the OpenShift cluster used for this course from the _Terminal_, run:


```
oc login -u developer -p developer
```

This will log you in using the credentials:

*   **Username:** `developer`
*   **Password:** `developer`

You should see the output:


```
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>
```


To create a new project called `myproject` run the command:


```
oc new-project myproject
```


You should see output similar to:


```
Now using project "myproject" on server "https://openshift:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app django-psql-example

to build a new example application in Python. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```


Switch to the _Console_ and login to the OpenShift web console using the same credentials you used above.


![alt_text](images/image21.png "image_tooltip")


This should leave you at the list of projects you have access to. As we only created the one project, all you should see is `myproject`.





![alt_text](images/image3.png "image_tooltip")


Click on `myproject` and you should then be at the _Overview_ page for the project. Select the _Developer_ perspective for the project instead of the _Adminstrator_ perspective in the left hand side menu. If necessary click on the hamburger menu icon top left of the web console to expose the left hand side menu.

As the project is currently empty, no workloads should be found and you will be presented with various options for how you can deploy an application.



![alt_text](images/image5.png "image_tooltip")



# Topic 2 - Deploying Using the Web Console

On the _Topology_ view, select _Container Image_. This should present the option of deploying an image by performing a search for the image on an image registry.

For this example, the application image we are going to deploy is being hosted on the Docker Hub Registry.

In the _Image name from external registry_ field enter:


```
openshiftkatacoda/blog-django-py
```


Press tab or click outside of the text field. This should trigger a query validate the image.



![alt_text](images/image7.png "image_tooltip")


From the name of the image, the _Application Name_ and deployment _Name_ fields will be automatically populated.

The deployment name is used in OpenShift to identify the resources created when the application is deployed. This will include the internal _Service_ name used by other applications in the same project to communicate with it, as well as being used as part of the default hostname for the application when exposed externally to the cluster via a _Route_.

The _Application Name_ field is used to group multiple deployments together under the same name as part of one overall application.

In this example leave both fields as their default values. For your own application you would consider changing these to something more appropriate.

At the bottom of this page you will see that the checkbox for creating a route to the application is selected. This indicates that the application will be automatically given a public URL for accessing it. If you did not want the deployment to be accessible outside of the cluster, or it was not a web service, you would de-select the option.

When you are ready, at the bottom of the page click on _Create_. This will return you to the _Topology_ view, but this time you will see a representation of the deployment, rather than the options for deploying an application.


<img src="images/image6.png" width="300"/>



You may see the color of the ring in the visualization change from white, to light blue and then blue. This represents the phases of deployment as the container for the application starts up.


# Topic 3 - Exploring the Topology View

To drill down and get further details on the deployment, click in the middle of the ring. This will result in a panel sliding out from the right hand side providing access to both an _Overview_:


<img src="images/image4.png" width="600"/>


and details on _Resources_ related to the deployment.




<img src="images/image1.png" width="600"/>


From the _Overview_ for the deployment, you can adjust the number of replicas, or pods, by clicking on the up and down arrows to the right of the ring.

The public URL for accessing the application can be found under _Resources_.

If you dismiss the panel, you can also access the application via its public URL, by clicking on the URL shortcut icon on the visualization of the deployment.


<img src="images/image2.png" width="300"/>


# Topic 4 - Deleting the Application 

Instead of deploying the existing container image from the web console, you can use the command line. Before we do that, lets delete the application we have already deployed.

To do this from the web console you could visit each resource type created and delete them one at a time. The simpler way to delete an application is from the command line using the `oc` program.

To see a list of all the resources that have been created in the project so far, you can run the command:


```
oc get all -o name
```


This will display output similar to:


```
pod/blog-django-py-1-cbp96
pod/blog-django-py-1-deploy
replicationcontroller/blog-django-py-1
service/blog-django-py
deploymentconfig.apps.openshift.io/blog-django-py
imagestream.image.openshift.io/blog-django-py
route.route.openshift.io/blog-django-py
```


You have only created one application, so you would know that all the resources listed will relate to it. When you have multiple applications deployed, you need to identify those which are specific to the application you may want to delete. You can do this by applying a command to a subset of resources using a label selector.

To determine what labels may have been added to the resources, select one and display the details on it. To look at the _Route_ which was created, you can run the command:


```
oc describe route/blog-django-py
```


This should display output similar to:


```
Name:                   blog-django-py
Namespace:              myproject
Created:                2 minutes ago
Labels:                 app=blog-django-py
                        app.kubernetes.io/component=blog-django-py
                        app.kubernetes.io/instance=blog-django-py
                        app.kubernetes.io/part-of=blog-django-py-app
Annotations:            openshift.io/generated-by=OpenShiftWebConsole
                        openshift.io/host.generated=true
Requested Host:         blog-django-py-myproject.2886795274-80-frugo03.environments.katacoda.com
                          exposed on router default (host apps-crc.testing) 2 minutes ago
Path:                   <none>
TLS Termination:        <none>
Insecure Policy:        <none>
Endpoint Port:          8080-tcp

Service:        blog-django-py
Weight:         100 (100%)
Endpoints:      10.128.0.205:8080
```


In this case when deploying the existing container image via the OpenShift web console, OpenShift has applied automatically to all resources the label `app=blog-django-py`. You can confirm this by running the command:


```
oc get all --selector app=blog-django-py -o name
```


This should display the same list of resources as when `oc get all -o name` was run. To double check that this is doing what is being described, run instead:


```
oc get all --selector app=blog -o name
```


In this case, because there are no resources with the label `app=blog`, the result will be empty.

Having a way of selecting just the resources for the one application, you can now schedule them for deletion by running the command:


```
oc delete all --selector app=blog-django-py
```


To confirm that the resources have been deleted, run again the command:


```
oc get all -o name
```


If you do still see any resources listed, keep running this command until it shows they have all been deleted. You can find that resources may not be deleted immediately as you only scheduled them for deletion and how quickly they can be deleted will depend on how quickly the application can be shutdown.

Although label selectors can be used to qualify what resources are to be queried, or deleted, do be aware that it may not always be the `app` label that you need to use. When an application is created from a template, the labels applied and their names are dictated by the template. As a result, a template may use a different labelling convention. Always use `oc describe` to verify what labels have been applied and use `oc get all --selector` to verify what resources are matched before deleting any resources.


#### **Topic 5 - Deploying Using the Command Line**

You now have a clean project again, so lets deploy the same existing container image, but this time using the `oc` command line program.

The name of the image you used previously was:


```
openshiftkatacoda/blog-django-py
```


If you have been given the name of an image to deploy and want to verify that it is valid from the command line, you can use the `oc new-app --search` command. For this image run:


```
oc new-app --search openshiftkatacoda/blog-django-py
```


This should display output similar to:


```
Docker images (oc new-app --docker-image=<docker-image> [--code=<source>])
-----
openshiftkatacoda/blog-django-py
  Registry: Docker Hub
  Tags:     latest
```


It confirms that the image is found on the Docker Hub Registry.

To deploy the image, you can run the command:


```
oc new-app openshiftkatacoda/blog-django-py
```


This will display out similar to:


```
--> Found container image 927f823 (4 months old) from Docker Hub for "openshiftkatacoda/blog-django-py"

    Python 3.5
    ----------
    Python 3.5 available as container is a base platform for building and running various Python 3.5 applications and frameworks. Python is an easy to learn, powerful programming language. It has efficient high-level data structures and a simple but effective approach to object-oriented programming. Python's elegant syntax and dynamic typing, together with its interpreted nature, make it an ideal language for scripting and rapid application development in many areas on most platforms.

    Tags: builder, python, python35, python-35, rh-python35

    * An image stream tag will be created as "blog-django-py:latest" that will track this image
    * This image will be deployed in deployment config "blog-django-py"
    * Port 8080/tcp will be load balanced by service "blog-django-py"
      * Other containers can access this service through the hostname "blog-django-py"

--> Creating resources ...
    imagestream.image.openshift.io "blog-django-py" created
    deploymentconfig.apps.openshift.io "blog-django-py" created
    service "blog-django-py" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/blog-django-py'
    Run 'oc status' to view your app.
```


OpenShift will assign a default name based on the name of the image, in this case `blog-django-py`. You can specify a different name to be given to the application, and the resources created, by supplying the `--name` option along with the name you wish to use as an argument.

Unlike how it is possible when deploying an existing container image from the web console, the application is not exposed outside of the OpenShift cluster by default. To expose the application created so it is available outside of the OpenShift cluster, you can run the command:


```
oc expose service/blog-django-py
```


Switch to the OpenShift web console by selecting on _Console_ to verify that the application has been deployed. Select on the URL shortcut icon displayed for the application on the _Topology_ view for the project to visit the application.

Alternatively, to view the hostname assigned to the route created from the command line, you can run the command:


```
oc get route/blog-django-py
```
