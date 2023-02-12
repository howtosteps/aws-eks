## Cleaning Up

For clean up, first ensure all applications are undeployed using 
```
kubectl delete -f <app-deployment-file>
kubectl delete -f <app-service-file>
```

First delete all nodegroups
```
eksctl delete nodegroup --cluster=EKS-course-cluster --name=scale-east1c
eksctl delete nodegroup --cluster=EKS-course-cluster --name=scale-east1d
eksctl delete nodegroup --cluster=EKS-course-cluster --name=scale-spot
```
Now delete the cluster
```
eksctl delete cluster --name=eks-cluster-ng
```