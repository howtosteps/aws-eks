# Create cluster with autoscaler

In this section, we will create a autoscaler that allows us to dynamically scale the nodes within a nodegroup - in and out.  We will start by adding 3 new nodegroups to our existing cluster `eks-cluster-ng`: 

* ***scale-east1c*** : Represents on-demand instances of one instance type running on a single AZ for stateful workloads
* ***scale-east1d*** : Represents on-demand instances of multiple instance types running on a single AZ for stateful workloads
* ***scale-spot*** : represents spot instances running on multi AZ for stateless workloads

The cluster auto-scaler automatically launches additional worker nodes if more resources are needed, and shutdowns worker nodes if they are under-utilized. The autoscaling works within a nodegroup, hence we will create a nodegroup first which has this feature enabled.

Create folder `cluster-autoscaler`. Copy the following contents into file  `eks-cluster.yaml`:

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-cluster-ng
  region: us-east-1

nodeGroups:
  - name: scale-east1c
    instanceType: t2.nano
    desiredCapacity: 1
    maxSize: 2
    availabilityZones: ["us-east-1c"]
    iam:
      withAddonPolicies:
        autoScaler: true
    labels:
      nodegroup-type: stateful-east1c
      instance-type: onDemand
  - name: scale-east1d
    instanceType: t2.nano
    desiredCapacity: 1
    maxSize: 3
    availabilityZones: ["us-east-1d"]
    iam:
      withAddonPolicies:
        autoScaler: true
    labels:
      nodegroup-type: stateful-east1d
      instance-type: onDemand
    ssh: # use existing EC2 key
      publicKeyName: eks-course
  - name: scale-spot
    desiredCapacity: 1
    maxSize: 4
    instancesDistribution:
      instanceTypes: ["t2.micro", "t2.small"]
      onDemandBaseCapacity: 0
      onDemandPercentageAboveBaseCapacity: 0
    availabilityZones: ["us-east-1a","us-east-1b"]
    iam:
      withAddonPolicies:
        autoScaler: true
    labels:
      nodegroup-type: stateless-workload
      instance-type: spot
    ssh: 
      publicKeyName: eks-course

availabilityZones: ["us-east-1a","us-east-1b","us-east-1c", "us-east-1d"]
```

This will tell EKS to :

* Apply additional cluster configurations to cluster `eks-cluster-ng`
* Apply defintions for 3 nodegroups :
    * scale-east1c : On-demand instances in us-east-1c
    * scale-east1d : On-demand instances in us-east-1d
    * scale-spot : Spot instances in us-east-1a, us-east-1b
* Note the autoScaler property is set to `true`

## Create cluster nodegroups
We will now create the new nodegroups in our existing cluster
```
eksctl create nodegroup --config-file=eks-course.yaml
```

```
kubectl get nodes
```
Let's delete the starter nodegroup `ng-1`
```
eksctl delete nodegroup --cluster=eks-cluster-ng --name=ng-1 --approve
```

## Create deployment for auto-scaler
Deploy the auto-scaler itself:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

## Add annotation to the deployment 
This prevents from being evicted
```
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```

## Set image version and cluster name 
We will now set matching image version and cluster name `eks-cluster-ng` in the deployment. Get the autoscaler image version:  

Open [Kubernetes/Autoscalar Releases](https://github.com/kubernetes/autoscaler/releases) and get the latest release version matching your Kubernetes version, e.g. Kubernetes 1.14 => check for 1.14.n where "n" is the latest release version

Edit deployment and set your EKS cluster name:

```
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```

* set the image version at property `image=k8s.gcr.io/cluster-autoscaler:vx.yy.z`  
* set your EKS cluster name at the end of property `- --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<<EKS cluster name>>`

```
kubectl -n kube-system describe deployment cluster-autoscaler
```