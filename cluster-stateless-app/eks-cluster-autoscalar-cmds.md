# create nodegroup 
eksctl create nodegroup --config-file=eks-course.yaml

# get nodegroup
eksctl get nodes

# delete cat e
# use original yaml file 
eksctl delete nodegroup -f eks-course.yaml --include='ng-1'
eksctl delete nodegroup -f eks-course.yaml --include='ng-1' --approve

# get nodes
kubectl get nodes

# auto-scaler
# DID NOT WORK -- get error
kubectl apply -f https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml 

# Update cluster to desired capacity of 0
eksctl upgrade cluster --config-file eks-course.yaml


# delete
# eksctl delete cluster --region=us-east-1 --name=EKS-course-cluster