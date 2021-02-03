Upgrade EKS to 1.18
https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html


1. kubectl -n kube-system set image deployment.apps/cluster-autoscaler=eu.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.18.3

2. aws eks --region eu-central-1 update-cluster-version --name qa --kubernetes-version 1.18

3. kubectl get daemonset kube-proxy --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}'

4. kubectl set image daemonset.apps/kube-proxy -n kube-system kube-proxy=6024017572.dkr.ecr.eu-central-1.amazonaws.com/eks/kube-proxy:v1.18.9-eksbuild.1

5. kubectl describe deployment coredns --namespace kube-system | grep Image | cut -d "/" -f 3    
5.1 kubectl get deployment coredns --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}' 
5.2 kubectl set image --namespace kube-system deployment.apps/coredns=6024857542.dkr.ecr.eu-central-1.amazonaws.com/eks/coredns:v1.7.0-eksbuild.1

6. When updating the CF stack, take into account that all nodes are in one Availability zone - eu-central-1b
Read before update CloudFormation stack https://medium.com/@trondhindenes/things-i-learned-about-cloudformation-autoscaling-groups-lately-ce03cd7f2c61

ami id ami-0a3d7ac8c4302b317 (find in CloudFormation and replace) https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html


Upgrade DNS (CoreDNS) to 1.7.0
Upgrade KubeProxy to 1.18.9 



Prepare and update cluster kubernetes to 1.18

https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html

When updating the CF stack, take into account that all nodes are in one Availability zone  -  eu-central-1b

NodeAutoScalingGroupDesiredCapacity 8

NodeAutoScalingGroupMaxSize 10

NodeAutoScalingGroupMinSize  7





Upgrade EKS to 1.16
https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html

kubectl get configmap coredns -n kube-system -o yaml |grep upstream

kubectl edit configmap coredns -n kube-system -o yaml (removing the line in the file that has the word upstream in it)

aws eks --region eu-central-1 update-cluster-version --name test-rancher-upgrade --kubernetes-version 1.16

kubectl get daemonset kube-proxy --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}'

kubectl set image daemonset.apps/kube-proxy -n kube-system kube-proxy=602401143452.dkr.ecr.eu-central-1.amazonaws.com/eks/kube-proxy:v1.16.13-eksbuild.1

kubectl describe daemonset aws-node --namespace kube-system | grep Image | cut -d "/" -f 2

curl -o aws-k8s-cni.yaml https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/v1.7.5/config/v1.7/aws-k8s-cni.yaml

sed -i -e 's/us-west-2/eu-central-1/' aws-k8s-cni.yaml

kubectl apply -f aws-k8s-cni.yaml

kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=eu.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.16.6

Read before update CloudFormation stack https://medium.com/@trondhindenes/things-i-learned-about-cloudformation-autoscaling-groups-lately-ce03cd7f2c61

ami id ami-0951ecbcce367c97b (find in CloudFormation and replace)
