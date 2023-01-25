# Simple cluster that auto-scales

# start
eksctl create cluster -f eks-cluster-ng.yaml

# get nodes
kubectl get nodes

# delete
eksctl delete cluster --region=us-east-1 --name=EKS-course-cluster


