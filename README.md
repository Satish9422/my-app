#Setup EKS Cluster on AWS
```bash
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-cluster
  region: us-east-1
  version: "1.29"

nodeGroups:
  - name: worker-node
    instanceType: t3.large
    desiredCapacity: 1
```
