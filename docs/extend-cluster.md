# Extend Cluster 
In this section we will extend the cluster by adding a nodegroup that contains a mix of:

* On-demand instances
* Spot instances

## Define Yaml file 
Create folder `cluster-ng-mixed`. We will now extend the cluster by adding a nodegroup to file `eks-cluster.yaml`. The file adds a new nodegroup `ng-mixed`. 

Copy the following contents :  

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-cluster-ng
  region: us-east-1

nodeGroups:
  - name: ng-1
    instanceType: t2.nano
    desiredCapacity: 2
    ssh: # use existing EC2 key
      publicKeyName: eks-course
  - name: ng-mixed
    minSize: 3
    maxSize: 5
    instancesDistribution:
      maxPrice: 0.2
      instanceTypes: ["t2.small", "t3.small"]
      onDemandBaseCapacity: 0
      onDemandPercentageAboveBaseCapacity: 50
    ssh: 
      publicKeyName: eks-course

```
This tells EKS to : 

* Add a new nodegroup `ng-mixed`
* Set min & max size
* Define the instance types
* 50% of the instance types are on-demand and the rest on spot 

## Add nodegroup  
First check your cluster is running :
```
eksctl get cluster
```
Let's add the new nodegroup :
```
eksctl create nodegroup --config-file=eks-cluster.yaml --include='ng-mixed'
```

## Delete nodegroup  
Now let's drain the `ng-mixed` nodegroup we just created so we don't incur additional costs :
```
eksctl delete nodegroup --config-file=eks-cluster.yaml --include='ng-mixed' --approve
```


