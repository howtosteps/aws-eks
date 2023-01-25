# Introduction
This tutorial describes how-to-steps to setup a simple EKS cluster. This tutorial will provide steps for following: 

* How to setup a simple kubernetes cluster using eksctl 
* How to setup kubetctl to access and manage your cluster 
* How to setup a stateless application
* How to setup a stateful application

## Application Structure
```
  |── docs                      # Contains edited nginx configuration file that will be copied to the image
  |    ├── img                  # Contains all images referenced in mkdocs
  |    ├── *.md                 # Other mkdocs .md files
  ├── mkdocs.yml                # YAML for for mkdocs
  ├── .gitattributes
  |
  ├── eks-cluster.yaml          # Yaml file for a simple cluster
  ├── eks-cluster-ng-mixed.yaml # Yaml file for cluster with different node groups
  ├── eks-cluster-cmds.md       # List of commands to run 
  ├── eks-cluster-ng-mixed.md   # List of commands to run 
  |
  ├── README.md                 # Standard README.md file
```


## Credits
Although I have leaned on information from various sources on the web ( including ChatGPT ), I did find this Udemy tutorial very useful in helping me craft this summarized how-to-steps :

[Amazon EKS Starter: Docker on AWS EKS with Kubernetes](https://www.udemy.com/course/amazon-eks-starter-kubernetes-on-aws/)