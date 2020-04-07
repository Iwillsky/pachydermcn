# pachydermcn

#Install awscli & tools
sudo pip install --upgrade awscli
sudo yum -y install jq gettext bash-completion
aws configure set default.region cn-northwest-1
aws s3 mb bucket <your-data-bucket-name>

#Install kubectl & enable bash_completion
sudo curl --silent --location -o /usr/local/bin/kubectl https://awshcls.s3.cn-northwest-1.amazonaws.com.cn/pachydermcn/amazon-eks-1.15.10/2020-02-22/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
kubectl version

#Install eksctl & enable bash_completion
curl --silent --location "https://awshcls.s3.cn-northwest-1.amazonaws.com.cn/pachydermcn/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
eksctl version

#Install aws-iam-authenticator
sudo curl --silent --location -o /usr/local/bin/aws-iam-authenticator https://awshcls.s3.cn-northwest-1.amazonaws.com.cn/pachydermcn/amazon-eks-1.15.10/2020-02-22/bin/linux/amd64/aws-iam-authenticator
sudo chmod +x /usr/local/bin/aws-iam-authenticator
aws-iam-authenticator version

#Install pachctl
curl -o /tmp/pachctl.tar.gz -L https://awshcls.s3.cn-northwest-1.amazonaws.com.cn/pachydermcn/pachctl.tar.gz 
tar -xvf /tmp/pachctl.tar.gz -C /tmp 
sudo cp /tmp/pachctl_1.10.0_linux_amd64/pachctl /usr/local/bin
pachctl version --client-only

#Validate IAM role    
aws sts get-caller-identity --query Arn | grep myEKSadmin -q && echo "IAM role valid" || echo "IAM role NOT valid"
   Prepare SSH Key

#Create keypair and import
ssh-keygen
input at first indication：/home/ec2-user/.ssh/pachnodekey
Press Enter*2 
aws ec2 import-key-pair --key-name "pachnodekey" --public-key-material file://~/.ssh/pachnodekey.pub
mv ~/.ssh/pachnodekey ~/.ssh/pachnodekey.pem
#如果node需要进一步调试可用pachnodekey.pem进行SSH登录

#Create EKS cluster
eksctl create cluster \
--name pachyderm-on-eks2 \
--region cn-northwest-1 \
--nodegroup-name standard-workers2 \
--node-type t3.medium \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3 \
--ssh-access \
--ssh-public-key pachngkey.pub \
--managed

kubectl get nodes

#Pachyderm要求节点组需要有S3操作的权限，待提示集群创建完成后，EKS控制台->Cluster->"pachyderm-on-eks"->Node Groups的详情里找到节点IAM名称


Referenc Document: EKS Handbook by Ryan Lao
