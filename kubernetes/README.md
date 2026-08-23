#Installation for Kubernetes:

#Install eksctl-
eksctl is a simple CLI tool for creating and managing Kubernetes clusters on Amazon EKS.
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Verify installation-
eksctl version

#Install kubectl-

Download kubectl-
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

#Verify kubectl-
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

#Install kubectl-
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Alternatively, install in local directory
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl

# Verify installation
kubectl version --client

#Download and Install AWS CLI-
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

#Configure AWS CLI-
aws configure

#Create an EKS Cluster-
eksctl create cluster \
  --name my-eks-cluster \
  --region ap-south-1 \
  --version 1.35 \
  --nodegroup-name my-nodes \
  --node-type t3.medium \
  --nodes 2

  #Update kubeconfig-
  aws eks update-kubeconfig --name my-eks-cluster

  #Delete the EKS Cluster-
  eksctl delete cluster --name my-eks-cluster --region ap-south-1


