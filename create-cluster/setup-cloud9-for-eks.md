---
created: 2022-05-21 12:46:05.435
last_modified: 2022-06-05 12:46:05.435
tags: aws/container/eks aws/cloud9 
---
```ad-attention
title: This is a github note

```
# setup-cloud9-for-eks

## install
1. resize disk - [[cloud9-resize-instance-volume-script]]
2. disable temporary credential from settings and delete `aws_session_token=` line in `~/.aws/credentials`
3. install dependencies
```sh
# install others
sudo yum -y install jq gettext bash-completion moreutils wget

# install kubectl with +/- 1 cluster version 1.23.8 / 1.22.11
# sudo curl --location -o /usr/local/bin/kubectl "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo curl --silent --location -o /usr/local/bin/kubectl "https://storage.googleapis.com/kubernetes-release/release/v1.22.11/bin/linux/amd64/kubectl"

# 1.22.x version of kubectl
# sudo curl --silent --location -o /usr/local/bin/kubectl "https://storage.googleapis.com/kubernetes-release/release/v1.22.11/bin/linux/amd64/kubectl"

sudo chmod +x /usr/local/bin/kubectl

kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
alias k=kubectl 
complete -F __start_kubectl k
echo "alias k=kubectl" >> ~/.bashrc
echo "complete -F __start_kubectl k" >> ~/.bashrc

# install awscli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# install eksctl
# consider install eksctl version 0.89.0
# if you have older version yaml 
# https://eksctl.io/announcements/nodegroup-override-announcement/
curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin
# wget https://github.com/weaveworks/eksctl/releases/download/v0.89.0/eksctl_Linux_amd64.tar.gz

eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion

# helm 3.8.2 (helm 3.9.0 will have issue #10975)
#curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
tar xf helm-v3.8.2-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
helm version --short

# install aws-iam-authenticator
curl -o aws-iam-authenticator https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo mv ./aws-iam-authenticator /usr/local/bin/

# option
# install jwt-cli
# https://github.com/mike-engel/jwt-cli/blob/main/README.md
sudo yum -y install cargo
cargo install jwt-cli

```

4. assign role to instance
```sh
ROLE_NAME=adminrole-$RANDOM
cat > trust.json <<-EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
EOF
aws iam create-role --role-name ${ROLE_NAME} \
  --assume-role-policy-document file://trust.json
aws iam attach-role-policy --role-name ${ROLE_NAME} \
  --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"
aws iam create-instance-profile --instance-profile-name ${ROLE_NAME}
aws iam add-role-to-instance-profile --instance-profile-name ${ROLE_NAME} --role-name ${ROLE_NAME}

INST_ID=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document |jq -r '.instanceId')
aws ec2 associate-iam-instance-profile --iam-instance-profile Name=${ROLE_NAME} --instance-id ${INST_ID}

```


## refer
- https://docs.amazonaws.cn/en_us/eks/latest/userguide/install-aws-iam-authenticator.html
- [[switch-role-to-create-dedicate-cloud9]]

## Turn off AWS managed temporary credentials 
[LINK](https://docs.aws.amazon.com/cloud9/latest/user-guide/security-iam.html#auth-and-access-control-temporary-managed-credentials)

If you turn off AWS managed temporary credentials, by default the environment cannot access any AWS services, regardless of the AWS entity who makes the request. If you can't or don't want to turn on AWS managed temporary credentials for an environment, but you still need the environment to access AWS services, consider the following alternatives:

- Attach an instance profile to the Amazon EC2 instance that connects to the environment. For instructions, see [Create and Use an Instance Profile to Manage Temporary Credentials](https://docs.aws.amazon.com/cloud9/latest/user-guide/credentials.html#credentials-temporary).
- Store your permanent AWS access credentials in the environment, for example, by setting special environment variables or by running the `aws configure` command. For instructions, see [Create and store permanent access credentials in an Environment](https://docs.aws.amazon.com/cloud9/latest/user-guide/credentials.html#credentials-permanent-create).





