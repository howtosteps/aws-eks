## Cleaning Up

For clean up, all applications are undeployed using 
```
kubectl delete -f <app-deployment-file>
kubectl delete -f <app-service-file>
```

Get all nodegroups
```
ksctl get nodegroup --cluster=eks-cluster-ng
```
```
PS C:\Users\aniru\workspace\github\aws-eks\cluster-stateless-app> eksctl get nodegroup --cluster=eks-cluster-ngCLUSTER         NODEGROUP       STATUS          CREATED                 MIN SIZE        MAX SIZE        DESIRED CAPACITY        INSTANCE TYPE   IMAGE ID           ASG NAME                                                                TYPE
eks-cluster-ng  scale-east1b    CREATE_COMPLETE 2023-02-14T12:10:31Z    1               2               1                       t2.nano         ami-0d4bdb1cf2f07d811      eksctl-eks-cluster-ng-nodegroup-scale-east1b-NodeGroup-PVRF5CRPR3DU     unmanaged
eks-cluster-ng  scale-east1c    CREATE_COMPLETE 2023-02-14T14:39:47Z    1               3               3                       t2.micro        ami-0d4bdb1cf2f07d811      eksctl-eks-cluster-ng-nodegroup-scale-east1c-NodeGroup-K1ALBP32WELN     unmanaged
eks-cluster-ng  scale-spot      CREATE_COMPLETE 2023-02-14T12:10:32Z    1               4               2                       t2.micro        ami-0d4bdb1cf2f07d811      eksctl-eks-cluster-ng-nodegroup-scale-spot-NodeGroup-1SZS5D6OYGK21      unmanaged
```

First delete all nodegroups
```
eksctl delete nodegroup --cluster=eks-cluster-ng --name=scale-east1b
eksctl delete nodegroup --cluster=eks-cluster-ng --name=scale-east1c
eksctl delete nodegroup --cluster=eks-cluster-ng --name=scale-spot
```
Now delete the cluster
```
eksctl delete cluster --name=eks-cluster-ng
```