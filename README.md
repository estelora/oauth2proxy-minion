# Using Kubernetes for Oauth2_proxy

In this tutorial we are going to set up a Kubernetes minion server that combines a basic guestbook app with oauth2_proxy.

###Kubernetes
Kubernetes is an open-source container orchestration and management system. This tutorial creates a Kubernetes minion.


### Let's get started.
> This tutorial assumes you have a functioning Kubernetes cluster. I did this in AWS.

#### Toolbox
* [Kubernetes](kubernetes.io)
* [Docker](https://www.docker.com/)
* [AWS](http://aws.amazon.com/)
* [kubectl CLI tool](https://coreos.com/kubernetes/docs/latest/configure-kubectl.html)
* [kube-aws CLI tool](https://coreos.com/kubernetes/docs/latest/kubernetes-on-aws.html)
* [oauth2_proxy + dependencies](https://github.com/bitly/oauth2_proxy)


#### Guide
This tutorial is based on the [Kubernetes guestbook example](https://github.com/kubernetes/kubernetes/tree/release-1.2/examples/guestbook). I have added a few [adjustments](https://github.com/estelora/oauth2proxy-minion) to this in a git repository, but for most of it, you can follow along with their documentation.


####1. Set up Redis Deployment and Service
   * Set Up Redis Master Deployment .yaml file
   * Set Up Redis Master Service .yaml file
   * Create Redis Master Service, the Redis Master     Deployment.

####2. Check Services, Deployments, and Pods 
   * Using kubectl, check your services, pods, and deployments. You can also check the logs of a single pod. The instructions on how to do this are in Kubernetes guestbook readme.

####3. Repeat Steps 1 & 2 for Redis Slave Service and Deployment, as well as the Front end Service and Deployment.

####4. Create an additional External Elastic Load Balancer in AWS (or somewhere else).
  * This allows external traffic into our minion.
  * I'll let you figure this one out. 
  * There is an appendix in the guestbook docs on this.
  
####5. Set up Oauth2_proxy Service
 * I assume you know something about oauth2_proxy. If not, read the [documentation](https://github.com/bitly/oauth2_proxy).
 * The .yaml files for oauth2_proxy can be found in my [git repository](https://github.com/estelora/oauth2proxy-minion) 
 * Set up an `oauth2proxy_service.yaml` file as follows:

```
apiVersion: v1
kind: Service
metadata:
  name: oauth2-proxy
  labels:
    app: oauth2-proxy
    tier: backend
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 4180
  selector:
    app: oauth2-proxy
    tier: backend   
```
####6. Create the Oauth2_proxy Service
 
####7. Set up Oauth2_proxy Deployment
 * I assume you know something about oauth2_proxy. If not, read the documentation.
 * Set up an `oauth2proxy_deployment.yaml` file as follows:
 
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: oauth2-proxy
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: oauth2-proxy
        tier: backend
    spec:
      containers:
      - name: oauth2-proxy
        # This sets the port at 4180
        image: estelora/oauth2_proxy
        volumeMounts:
        - mountPath: /etc/nginx/conf.d
          name: nginx-volume
        ports:
        - containerPort: 4180
        command:
        # Please set these variables for your project.
        - oauth2_proxy
        # Here is an example of service discovery.
        - --upstream=http://frontend
        - --email-domain=example.com
        - --client-id=client-id.apps.googleusercontent.com
        - --client-secret="google-client-secret"
        - --cookie-domain=.internal.example.com
        - --redirect-url=https://auth.internal.example.com/oauth2/callback
        - --cookie-secret="secret-chocolate-chip-cookie"
        # This variable stays the same - this is an internal IP
        - --http-address=0.0.0.0:4180
```
 
 * The port is set to 4180 in the container, service, and deployment. 
 * The `command:` sets up oauth2_proxy command line arguments.
 * The upstream shows Kubernetes' service discovery - the internal address is `http://frontend`.
8. Create the Oauth2_proxy Service
9. Debug Network Issues as necessary!
  * You can adjust your network with .yaml on the Kubernetes side. 
  * You can use `kubectl` logs and ssh into a docker container itself.
  * Adjust firewalls for both your services and external load balancers.


##### Kuberenetes Glossary
* **Deployment:** set of parameters for the desired state of pods
* **Minion:** a server that performs work, configures networking for containers, and runs tasks assigned to containers.
* **Service:** internal load balancer
* **Node:** provisioned hardware (in this case, a VM in the cloud).
* **Pod:** container or group of containers that support each other to run tasks.
* **Service Discovery:** allows you to hard-code host names within your Kubernetes minion into your code.
* Containers: a Docker container or google container.


