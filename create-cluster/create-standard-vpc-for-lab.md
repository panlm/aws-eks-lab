---
created: 2022-04-10 22:12:29.404
last_modified: 2022-04-10 22:12:29.404
tags: aws/network/vpc aws/mgmt/cloudformation 
---
```ad-attention
title: This is a github note

```
## using cloudformation template 

```sh
region_name=us-east-1
bucket_name=$(aws s3 mb s3://panlm-$RANDOM-$RANDOM |awk '{print $2}')

wget -O aws-vpc.template.yaml https://github.com/panlm/quickstart-aws-vpc/raw/main/templates/aws-vpc.template.yaml
aws s3 cp aws-vpc.template.yaml s3://${bucket_name}/

aws cloudformation create-stack --stack-name stack1-$RANDOM --parameters ParameterKey=AvailabilityZones,ParameterValue="${region_name}a\,${region_name}b" --template-url https://${bucket_name}.s3.amazonaws.com/aws-vpc.template.yaml

```

- no s3 endpoint
- security group named eks-shared-sg
- tag subnet `kubernetes.io/role/internal-elb` / `kubernetes.io/role/elb`
- verify in china region

## refer
- [[cloudformation-cli]]



