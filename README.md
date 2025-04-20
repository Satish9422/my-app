# Jenkins on kubernetes with blue-green deployment
## Setup EKS Cluster on AWS
```bash
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-eks-clusters
  region: us-east-1
  version: "1.29"

nodeGroups:
  - name: worker-node
    instanceType: t3.large
    desiredCapacity: 1
```
### Create IAM policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --cluster my-eks-clusters --approve
```

### Create Service Account on cluster for EBS Driver
```bash
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster my-eks-clusters \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-name jenkins-eks-role
```
### Create EBS volume on AWS and attach to persistent volume in cluster.
```bash kubectl apply -f infra/jenkins-pv.yaml
kubectl apply -f infra/jenkins-pvc.yaml
```

### Create Service Account for jenkins to allow access to cluster resources
```bash kubectl apply -f infra/jenkins-sa.yaml
```

### Create role binding for jenkins Service Account
```bash kubectl apply -f infra/rbac.yaml
```

### Create Jenkins pod on kubernetes 
```bash kubectl apply -f infra/jenkins.yaml
```




