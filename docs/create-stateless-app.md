# Create Stateless App
In this section we will :

* Deploy backend resources
* Deploy frontend resources
* Scale pods up/down 
* Perform chaos testing

## Setup sample guestbook app
This example is based on [Kubernetes Guestbook](https://github.com/kubernetes/examples/tree/master/guestbook). The application consists of:

* A Redis backend with a single leader pod for Writes & multiple follower pods for Reads
* A frontend app ( Guestbook app ) in PHP loadbalanced to public using a load balancer
* Guestbook app reads load balanced to multiple redis followers (reads)
* Guestbook app writes directed towards single redis leader (writes)

![Screenshot](img/eks-stateless-app.png)

## Backend Deployment
Our configuration files for Redis leader/followers includes both Deployment and Services kinds. Deployments and Services are often used in tandem: Deployments working to define the desired state of the application and Services working to make sure communication between almost any kind of resource and the rest of the cluster is stable and adaptable. 

For the purpose of this application we will scale our nodegroup `scale-spot` to 3 instances
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> eksctl scale nodegroup --cluster=eks-cluster-ng --nodes=3 --nodes-max=4 --name=scale-spot
2023-02-14 10:22:58 [ℹ]  scaling nodegroup "scale-spot" in cluster eks-cluster-ng
2023-02-14 10:23:02 [ℹ]  nodegroup successfully scaled
```
### Redis leader
For the Redis leader, first create the Deployment file `redis-leader-deployment.yaml`. Copy these contents to it : 
```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: leader
        tier: backend
    spec:
      containers:
      - name: leader
        image: "docker.io/redis:6.0.5"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```
This tells EKS :

* The deployment has replica of 1
* Specifies the source of image 
* Set resource requirement for cpu & memory
* Set the container port to 6379

Now let's create the Service on top of your deployment. Create file `redis-leader-service.yaml` and copy the following contents to it:
```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: v1
kind: Service
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: leader
    tier: backend
```
Notice that the metadata name and labels are same as the redis-leader Deployment. Here the target port that is exposed is set to 6379. 

Now that the configuration files are defined, let's deploy the Redis leader pod :
```
kubectl apply -f redis-leader-deployment.yaml
```
Let's test by checking if the pods were created
```
kubectl get pods
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pod 
NAME                               READY   STATUS    RESTARTS   AGE
redis-leader-766465cd9c-wh94g      1/1     Running   0          46s
test-autoscaler-6545fc8df4-7pktn   0/1     Pending   0          66m
```

Now we apply Redis service on top of this :
```
kubectl apply -f redis-leader-service.yaml
```
Let's test using :
```
kubectl get service redis-leader
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get service redis-leader
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
redis-leader   ClusterIP   10.100.232.230   <none>        6379/TCP   11s
```
### Redis follower
For the Redis follower, let's create the Deployment file `redis-followers-deployment.yaml`. Copy these contents to it : 
```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: follower
        tier: backend
    spec:
      containers:
      - name: follower
        image: gcr.io/google_samples/gb-redis-follower:v2
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```
Just like before we will now create the Service file as `redis-follower-service.yaml`. Copy the following contents to it : 
```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    app: redis
    role: follower
    tier: backend
```
Notice the labels that match the Deployment file and the internal port that the redis follower service should run on. 

Now that the configuration files are defined, let's deploy the Redis follower pod :
```
kubectl apply -f redis-followers-deployment.yaml
kubectl apply -f redis-follower-service.yaml
```
Let's check the status of Redis followers by running : 
```
kubectl get pods -o wide
```
```
kubectl get service
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get service
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes       ClusterIP   10.100.0.1       <none>        443/TCP    7h40m
redis-follower   ClusterIP   10.100.249.155   <none>        6379/TCP   41m
redis-leader     ClusterIP   10.100.232.230   <none>        6379/TCP   42m
```
```
kubectl describe node <node-name>
```
## Frontend app
Now we will deploy the frontend app that will consist of 

* 3 replicas of the Guestbook App
* ELB LoadBalancer service

### Guestbook deployment
We will start with defining the Deployment file for the frontend app. Create `frontend-deployment.yaml` and copy the following contents : 
```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
        app: guestbook
        tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v5
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 80
```
This tells EKS to :

* Generate 3 replicas of the frontend app
* Specfiy image source
* Set the container resources
* Set the container port

Let's apply the deployment 
```
kubectl apply -f frontend-deployment.yaml
```
Check all the pods for the guestbook app
```
kubectl get pods
kubectl get pods -l app=guestbook
kubectl get pods -l app=guestbook -l tier=frontend
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pods -l app=guestbook -l tier=frontend
NAME                        READY   STATUS    RESTARTS   AGE
frontend-57df59b89c-5rkzr   1/1     Running   0          28m
frontend-57df59b89c-f26lm   1/1     Running   0          28m
frontend-57df59b89c-s7n2s   1/1     Running   0          28m
```

### Loadbalancer service
We will now apply the loadbalancer service that allows us to expose all the loadbalanced service to the public. 

Create file `frontend-service.yaml` and copy the following : 
```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  type: LoadBalancer
  ports:
    # the port that this service should serve on
  - port: 80
  selector:
    app: guestbook
    tier: frontend
```

Let's apply the loadbalanced service
```
kubectl apply -f frontend-service.yaml
```

Check all services
```
kubectl get services
kubectl get service frontend 
kubetctl get pods -o wide
kubectl describe service frontend
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get service frontend
NAME       TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)        AGE
frontend   LoadBalancer   10.100.141.207   ab8f47054a1bb400f8700134e4410f1c-1044573003.us-east-1.elb.amazonaws.com   80:30074/TCP   91s
```
Finally check AWS mgmt console for the ELB which has been created !

### Access from outside the cluster

Grab the public DNS of the frontend service LoadBalancer (ELB):
```
kubectl describe service frontend
```
Copy the name and paste it into your browser !!!

## Scale Pods up and down
We can scale pods up and down in the following ways:

* kubectl
* Update YAML file

### Using kubectl
We can scale up and down using kubectl command. To scale up run the following command :  
```
kubectl scale deployment frontend --replicas=5
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
frontend-57df59b89c-5rkzr          1/1     Running   0          37m
frontend-57df59b89c-f26lm          1/1     Running   0          37m
frontend-57df59b89c-rhwrv          0/1     Pending   0          5s
frontend-57df59b89c-s7n2s          1/1     Running   0          37m
frontend-57df59b89c-xxjll          1/1     Running   0          5s
redis-follower-84fcc94dfc-2k2pn    1/1     Running   0          91m
redis-follower-84fcc94dfc-k6d2t    1/1     Running   0          91m
redis-leader-766465cd9c-wh94g      1/1     Running   0          93m
test-autoscaler-59c78f9974-7jn9n   1/1     Running   0          76m
```

Let's check the pods now
```
kubectl get pods
```
Now let us scale it down
```
kubectl scale deployment frontend --replicas=3
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
frontend-57df59b89c-5rkzr          1/1     Running   0          38m
frontend-57df59b89c-f26lm          1/1     Running   0          38m
frontend-57df59b89c-s7n2s          1/1     Running   0          38m
redis-follower-84fcc94dfc-2k2pn    1/1     Running   0          92m
redis-follower-84fcc94dfc-k6d2t    1/1     Running   0          92m
redis-leader-766465cd9c-wh94g      1/1     Running   0          94m
test-autoscaler-59c78f9974-7jn9n   1/1     Running   0          76m
```
You can check pods are terminating by checking the pods
```
kubectl get pods
```
### Using YAML file
We can scale the pods by updating the replicas tag in our yaml files. Let's test this out by updating file `redis-followers-deployment.yaml`. Update the replicas count to 5 as follows :

```
# SOURCE: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 5
  selector:
    matchLabels:
        app: guestbook
        tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v5
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 80
```
Let's apply the new deployment file
```
kubectl apply -f redis-followers-deployment.yaml
```
Let's check this by running
```
kubectl get deployment redis-follower
kubectl get pods
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get deployment redis-follower
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
redis-follower   5/5     5            5           99m
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
frontend-57df59b89c-5rkzr          1/1     Running   0          42m
frontend-57df59b89c-f26lm          1/1     Running   0          42m
frontend-57df59b89c-s7n2s          1/1     Running   0          42m
redis-follower-84fcc94dfc-2k2pn    1/1     Running   0          95m
redis-follower-84fcc94dfc-5fvt6    1/1     Running   0          31s
redis-follower-84fcc94dfc-k6d2t    1/1     Running   0          95m
redis-follower-84fcc94dfc-s6mcd    1/1     Running   0          31s
redis-follower-84fcc94dfc-v4bcx    1/1     Running   0          31s
redis-leader-766465cd9c-wh94g      1/1     Running   0          98m
test-autoscaler-59c78f9974-7jn9n   1/1     Running   0          80m
```
## Perform chaos testing
We will demonstrate self-healing mechanism of K8s by:

* Kill pods
* Stop K8s worker node(s)
```
kubectl get pods
kubectl get nodes
```
```
kubectl get pods -o wide
```
Now let's delete one of front-end pods
```
kubectl delete pod <frontend-pod>
```
Now check for pods look a the age column. Kubernetes restarts the pod
```
kubectl get pods -o wide
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl delete pod redis-follower-84fcc94dfc-k6d2t
pod "redis-follower-84fcc94dfc-k6d2t" deleted
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
frontend-57df59b89c-5rkzr          1/1     Running   0          54m
frontend-57df59b89c-f26lm          1/1     Running   0          54m
frontend-57df59b89c-s7n2s          1/1     Running   0          54m
redis-follower-84fcc94dfc-2k2pn    1/1     Running   0          107m
redis-follower-84fcc94dfc-4wn4v    1/1     Running   0          87s
redis-follower-84fcc94dfc-5fvt6    1/1     Running   0          12m
redis-follower-84fcc94dfc-s6mcd    1/1     Running   0          12m
redis-follower-84fcc94dfc-v4bcx    1/1     Running   0          12m
redis-leader-766465cd9c-wh94g      1/1     Running   0          110m
test-autoscaler-59c78f9974-7jn9n   1/1     Running   0          92m
```
Now let's use AWS console to  stop any instance ( AWS > EC2 > instances). You will notice 2 things :

* Kubernetes will try to recover the nodes
* Kubernetes will reshuffle the nodes if required 

## Cleaning up

Run the following commands to save costs
```
kubectl delete deployment -l app=redis
kubectl delete service -l app=redis
kubectl delete deployment frontend
kubectl delete service frontend
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
test-autoscaler-59c78f9974-7jn9n   1/1     Running   0          97m
```