---
created: 2022-05-21 12:43:38.021
last_modified: 2022-05-21 12:43:38.021
tags: aws/container/eks 
---
```ad-attention
title: This is a github note

```
# eks-public-access-cluster

## prep
- do not need to create vpc in advance
- [[setup-cloud9-for-eks]] or using your local environment

## cluster yaml
- dont put `subnets`/`sharedNodeSecurityGroup` in your `vpc` section. eksctl will create a clean vpc for you
- dont use `privateCluster` section, you could make cluster api server endpoint `public` or `public and private`
- you still could put your group node in private subnet for security consideration
- recommend for most of poc environment

```yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ekscluster1 # MODIFY cluster name, have another one in nodeGroup section
  region: "us-east-1" # MODIFY region
  version: "1.21" # MODIFY version

availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"]

# REPLACE THIS CODE BLOCK
# vpc:
#   subnets:
#     private:
#       us-east-2a:
#         id: subnet-xxxxxxxx
#       us-east-2b:
#         id: subnet-xxxxxxxx
#     public:
#       us-east-2a:
#         id: subnet-xxxxxxxx
#       us-east-2b:
#         id: subnet-xxxxxxxx
#   sharedNodeSecurityGroup: sg-xxxxxxxx
vpc:
  clusterEndpoints:
    privateAccess: true
    publicAccess: true

cloudWatch:
  clusterLogging:
    enableTypes: ["*"]

# secretsEncryption:
#   keyARN: ${MASTER_ARN}

managedNodeGroups:
- name: managed-ng
  minSize: 1
  maxSize: 5
  desiredCapacity: 1
  instanceType: m5.large
  ssh:
    enableSsm: true
  privateNetworking: true

nodeGroups:
- name: ng1
  minSize: 1
  maxSize: 5
  desiredCapacity: 1
  instanceType: m5.large
  ssh:
    enableSsm: true
  privateNetworking: true
  ami: ami-06a8057d9b6a06ee6
  amiFamily: AmazonLinux2
  overrideBootstrapCommand: |
    #!/bin/bash
    source /var/lib/cloud/scripts/eksctl/bootstrap.helper.sh
    /etc/eks/bootstrap.sh ${CLUSTER_NAME} --container-runtime containerd --kubelet-extra-args "--node-labels=${NODE_LABELS}"

iam:
  withOIDC: true

addons:
- name: vpc-cni # no version is specified so it deploys the default version
  version: latest

```


## default tags on subnet
[[eksctl-default-tags-on-subnet]]

## network topo preview
- [[security-group-for-eks-deepdive]]

## refer
- [[eks-private-access-cluster]]
- [[eks-nodegroup]]



