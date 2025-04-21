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
```bash
kubectl apply -f infra/jenkins-pv.yaml
kubectl apply -f infra/jenkins-pvc.yaml
```

### Create Service Account for jenkins to allow access to cluster resources
```bash
kubectl apply -f infra/jenkins-sa.yaml
```

### Create role binding for jenkins Service Account
```bash
kubectl apply -f infra/rbac.yaml
```

### Create Jenkins pod on kubernetes 
```bash
kubectl apply -f infra/jenkins.yaml
```
### Configure and Install Jenkins

Install and configure Kubernetes plugin

Create jenkins agent pod template in kubernetes cloud.

Use jnlp, kaniko and kubectl to spun up docker container on jenkins agent pod.

### Create Jenkinsfile 
create jenkinsfile utilising kaniko and kubectl docker images for agent pod to run on eks cluster.

### Deploy application 
```bash
kubectl aaply -f k8s/blue-deployment.yaml
kubectl apply -f k8s/service.yaml
```
### Push new image to docker registry.
Create sercret in eks cluster with docker credentials
```bash 
kubectl create secret docker-registry regcred \
--docker-server=https://index.docker.io/v1/ \
--docker-username=dokcer_username \
--docker-password=<docker access token> \
--docker-email=docker_email

### Blue-Green Deployment 
```bash
kubectl apply -f k8s/green-deployment.yaml

### Check green deployment status
```bash
kubectl rollout status deployment/myapp-green
```
       
### Switch Traffic to green deployment 
If green deployment status is successful switch traffic to it.
```bash 
    kubectl patch service myapp-service -p \'{"spec":{"selector":{"app":"myapp-green"}}}\'
```
### Cleanup Blue Deployment 
```bash
kubectl delete deployment myapp-blue
```




