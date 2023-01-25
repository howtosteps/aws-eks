# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

```
PS C:\Users\aniru\workspace\github\aws-eks> kubectl get nodes
Kubeconfig user entry is using deprecated API version client.authentication.k8s.io/v1alpha1. Run 'aws eks update-kubeconfig' to update.
NAME                             STATUS   ROLES    AGE   VERSION
ip-192-168-21-105.ec2.internal   Ready    <none>   71m   v1.22.15-eks-fb459a0
ip-192-168-54-187.ec2.internal   Ready    <none>   71m   v1.22.15-eks-fb459a0
```

```
S C:\Users\aniru\workspace\github\aws-eks> eksctl get nodegroup --cluster eks-cluster-ng
CLUSTER         NODEGROUP       STATUS          CREATED                 MIN SIZE        MAX SIZE        DESIRED CAPACITY        INSTANCE TYPE   IMAGE ID                ASG NAME                                       TYPE
eks-cluster-ng  ng-1            CREATE_COMPLETE 2023-01-25T00:22:41Z    2               2               2                       t2.nano         ami-03a30cc1dda93f173   eksctl-eks-cluster-ng-nodegroup-ng-1-NodeGroup-6387ICJKD3BS     unmanaged
```