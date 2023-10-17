# Creating a Kubernetes Pod from Docker Image

{% hint style="info" %}
You will need the below details from your instructor

1. Kubernetes or Openshift API URL: [**https://api.swapnil.mask365.com:6443**](https://api.swapnil.mask365.com:6443)
2. Username:&#x20;
3. Password: **redhat**
{% endhint %}

1. Install openshift and kubernetes client utility

```
[student@servera ~]$ wget https://github.com/openshift/okd/releases/download/4.6.0-0.okd-2020-12-12-135354/openshift-client-linux-4.6.0-0.okd-2020-12-12-135354.tar.gz
```

```
[student@servera ~]$ tar xvf openshift-client-linux-4.6.0-0.okd-2020-12-12-135354.tar.gz
```

```
[student@servera ~]$ mkdir ~/bin
[student@servera ~]$ mv oc kubectl ~/bin/
```

2\. Login to openshift cluster using the details provided to you

```
[student@servera ~]$ oc login
Server [https://localhost:8443]: https://api.swapnil.mask365.com:6443
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n): y

Authentication required for https://api.swapnil.mask365.com:6443 (openshift)
Username: your_user_name
Password: redhat
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

Welcome! See 'oc help' to get started.
```

3\. Create a project named yourusername-testimage

{% hint style="info" %}
replace yourusername with the username provided to you
{% endhint %}

```
[student@servera ~]$ oc new-project yourusername-testproject


Now using project "user15-testproject" on server "https://api.swapnil.mask365.com:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname


```

4\. create an application using docker image

```
[student@servera ~]$ oc new-app --name=hello --docker-image=quay.io/jswapnil/pythontest

--> Found container image 0341bc7 (10 months old) from quay.io for "quay.io/jswapnil/pythontest"

    * An image stream tag will be created as "hello:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "hello" created
    deployment.apps "hello" created
    service "hello" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/hello' 
    Run 'oc status' to view your app.
[student@servera ~]$ 

```

5\. wait for the pod to be in running state

```
[student@servera ~]$ oc get pods
NAME                     READY   STATUS    RESTARTS   AGE
hello-7b497df95d-j2tl2   1/1     Running   0          49s
```

6\. inspect the service and create a route

```
[student@servera ~]$ oc get service
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hello   ClusterIP   172.30.156.12   <none>        8080/TCP   2m13s


[student@servera ~]$ oc expose service hello
route.route.openshift.io/hello exposed



[student@servera ~]$ oc get route
NAME    HOST/PORT                                           PATH   SERVICES   PORT       TERMINATION   WILDCARD
hello   hello-user15-testproject.apps.swapnil.mask365.com          hello      8080-tcp                 None


```

{% hint style="info" %}
in this case, the route is hello-user15-testproject.apps.swapnil.mask365.com, it might be different for you
{% endhint %}

7\. to access the application open the route in your browser or use curl

8\. delete the pod and try accessing the route again after 5 to 10 seconds

{% hint style="info" %}
note that the pod name will be different for you, to check the pod name use the below command

**`oc get pods`**
{% endhint %}

```
[student@servera ~]$ oc get pods
NAME                     READY   STATUS    RESTARTS   AGE
hello-7b497df95d-j2tl2   1/1     Running   0          6m52s


[student@servera ~]$ oc delete pod hello-7b497df95d-j2tl2
pod "hello-7b497df95d-j2tl2" deleted


[student@servera ~]$ oc get route
NAME    HOST/PORT                                           PATH   SERVICES   PORT       TERMINATION   WILDCARD
hello   hello-user15-testproject.apps.swapnil.mask365.com          hello      8080-tcp                 None


[student@servera ~]$ curl http://hello-user15-testproject.apps.swapnil.mask365.com
<h1>Welcome to Kubernetes</h1>
```

9\. application is still accessible as the controller recreated the pod. Next delete the application

```
[student@servera ~]$ oc delete all -l app=hello
service "hello" deleted
deployment.apps "hello" deleted
imagestream.image.openshift.io "hello" deleted
route.route.openshift.io "hello" deleted
[student@servera ~]$
```
