#Installation for Kubernetes:

sudo apt update -y

sudo apt install -y curl tar unzip git

#eksctl

curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
--output /tmp/eksctl.tar.gz

tar -xzf /tmp/eksctl.tar.gz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin/eksctl

sudo chmod +x /usr/local/bin/eksctl

eksctl version

#kubectl

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client

rm -f kubectl kubectl.sha256

#AWS CLI

sudo apt install -y unzip curl

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip -q awscliv2.zip

sudo ./aws/install

aws --version

rm -rf aws awscliv2.zip

#Configure AWS

aws configure
Verify AWS
aws sts get-caller-identity

#Create EKS

eksctl create cluster \
  --name my-eks-cluster \
  --region ap-south-1 \
  --version 1.35 \
  --nodegroup-name my-nodes \
  --node-type t3.medium \
  --nodes 2
  
#Configure kubectl

aws eks update-kubeconfig \
  --region ap-south-1 \
  --name my-eks-cluster
  
#Verify

kubectl get nodes
kubectl get pods -A
kubectl cluster-info

Delete
eksctl delete cluster \
  --name my-eks-cluster \
  --region ap-south-1


