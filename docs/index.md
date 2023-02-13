# Introduction
This tutorial describes how-to-steps to setup a simple EKS cluster. This tutorial will provide steps for following: 

* How to setup a simple kubernetes cluster using eksctl 
* How to add new nodegroups
* How to add autoscaler to dynamically scale in and out 
* How to setup a stateless application

## Application Structure
```
  |── docs                                  # Folder contains all the mkdocs files
  |    ├── img                              # Contains all images referenced in mkdocs
  |    ├── *.md                             # Other mkdocs .md files
  ├── mkdocs.yml                            # Yaml for for mkdocs
  ├── .gitattributes
  |
  |── cluster-ng                            # Folder containing a simple EKS cluster with on-demand instances
  |    ├── eks-cluster.yaml                 # Yaml file for a simple cluster
  |    ├── eks-cluster-cmds.md              # List of commands to run 
  |── cluster-ng-mixed                      # Folder containing an EKS cluster with spot & on-demand instances
  |    ├── eks-cluster.yaml                 # Yaml file for cluster with different node groups
  |── cluster-autoscaler                    # Folder containing a EKS cluster with auto-scaler
  |    ├── eks-cluster.yaml                 # Yaml file for a simple cluster
  |    ├── eks-cluster-cmds.md              # List of commands to run 
  |── cluster-stateless-app                 # Folder containing a simple stateless example from kubernetes.io
  |    ├── frontend-deployment.yaml         # Yaml file for frontend deployment
  |    ├── frontend-service.yaml            # Yaml file for frontend service
  |    ├── redis-leader-deployment.yaml     # Yaml file for frontend deployment
  |    ├── redis-leader-service.yaml        # Yaml file for frontend service
  |    ├── redis-followers-deployment.yaml  # Yaml file for frontend deployment
  |    ├── redis-follower-service.yaml      # Yaml file for frontend service
  |    ├── cluster-stateless-app-cmds.md    # List of commands to run
  ├── README.md                             # Standard README.md file
```


## Credits
Although much of the material has been sources from various sources on the web ( including ChatGPT ), I did find this Udemy tutorial very useful in crafting these how-to-steps :

[Amazon EKS Starter: Docker on AWS EKS with Kubernetes](https://www.udemy.com/course/amazon-eks-starter-kubernetes-on-aws/)