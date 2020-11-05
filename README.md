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
