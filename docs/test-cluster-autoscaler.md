# Test Auto-scaler with Nginx deployment
We will now test the auto-scaler and scale it up and down

## Create the deployment file
In folder `cluster-autoscaler`, we will create a new deployment file `nginx-deployment.yaml`. Copy the following contents to it: 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        service: nginx
        app: nginx
    spec:
      containers:
      - image: nginx
        name: test-autoscaler
        resources:
          limits:
            cpu: 300m
            memory: 256Mi
          requests:
            cpu: 300m
            memory: 256Mi
      nodeSelector:
        instance-type: spot
```
This will tell eksctl to :

* Refer to this one as of kind: Deployment
* This will have 1 replica with service: nginx and app: nginx
* Container will use image: nginx and will call it as name: test-autoscaler
* Specify resources with cpu & memory tags
* nodeSelector specifies what kind of instances we will need using tag instance-type: spot

## Deploy Nginx

We will now apply the deployment file using kubectl command
```
kubectl apply -f nginx-deployment.yaml
```
Now check if the pod is running
```
kubectl get pod
```
```
S C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
test-autoscaler-6545fc8df4-7pktn   0/1     Pending   0          22s
```
check the instance types 
```
kubectl get nodes -l instance-type=spot
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl get nodes -l instance-type=spot
NAME                            STATUS   ROLES    AGE    VERSION
ip-192-168-12-53.ec2.internal   Ready    <none>   8m5s   v1.23.15-eks-49d8fe8
```
## Scale the deployment
Let's scale deployment to 3 replicas so we can test the autoscaler
```
kubectl scale --replicas=3 deployment/test-autoscaler
```
```
S C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl scale --replicas=3 deployment/test-autoscaler
deployment.apps/test-autoscaler scaled
```
You should see 3 pods running now but using the same node 
```
kubectl get pod
```
```
S C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
test-autoscaler-6545fc8df4-6qk5f   0/1     Pending   0          33s
test-autoscaler-6545fc8df4-7pktn   0/1     Pending   0          6m12s
test-autoscaler-6545fc8df4-hmrxd   0/1     Pending   0          33s
```
```
kubectl get nodes -l instance-type=spot
``` 
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl get nodes -l instance-type=spot
NAME                            STATUS   ROLES    AGE   VERSION
ip-192-168-12-53.ec2.internal   Ready    <none>   15m   v1.23.15-eks-49d8fe8
```
### Scale-out the deployment
Let's test scale-out by increasing the number of replicas to 4
```
kubectl scale --replicas=4 deployment/test-autoscaler
```
Now check the pods 
```
kubectl get pods -o wide
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl get pods -o wide --watch
NAME                               READY   STATUS    RESTARTS   AGE     IP       NODE     NOMINATED NODE   READINESS GATES
test-autoscaler-6545fc8df4-6qk5f   0/1     Pending   0          4m39s   <none>   <none>   <none>           <none>
test-autoscaler-6545fc8df4-7pktn   0/1     Pending   0          10m     <none>   <none>   <none>           <none>
test-autoscaler-6545fc8df4-hmrxd   0/1     Pending   0          4m39s   <none>   <none>   <none>           <none>
test-autoscaler-6545fc8df4-kfqg4   0/1     Pending   0          13s     <none>   <none>   <none>           <none>
```

And let's check the # of instances. It should increase to accomodate the increased # of replicas
```
kubectl get nodes -l instance-type=spot
``` 
Now check page `Spot Requests` on AWS Console after a few minutes

### Scale-in the deployment
Let's test scale-out by reducing the number of replicas to 1
```
kubectl scale --replicas=1 deployment/test-autoscaler
```
Now check the pods 
```
kubectl get pod
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl scale --replicas=1 deployment/test-autoscaler 
deployment.apps/test-autoscaler scaled
PS C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
test-autoscaler-6545fc8df4-7pktn   0/1     Pending   0          21m
```

And let's check the # of instances. It should increase to accomodate the increased # of replicas
```
kubectl get nodes -l instance-type=spot
``` 
Now check page `EC2 > Instances > Spot Requests` on AWS Console after a few minutes. Also check page `EC2 > Auto Scaling Groups`

### Check logs

```
kubectl -n kube-system logs deployment.apps/cluster-autoscaler
```
```
S C:\Users\aniru\workspace\github\aws-eks\cluster-autoscaler> kubectl -n kube-system logs deployment.apps/cluster-autoscaler
Found 2 pods, using pod/cluster-autoscaler-c5bf8cc8-glhlr
```

