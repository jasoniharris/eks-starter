eks-starter



# Run AWS CLI via Docker
docker pull amazon/aws-cli

# Run Amazon CLI
docker run -it --rm -v ${HOME}/projects/docker-development-youtube-series:/work -w /work --entrypoint /bin/sh aws_cli_harrzjas:latest
docker run -it --rm --entrypoint /bin/sh aws_cli_harrzjas:latest

yum install -y jq gzip nano tar git

# Set up AWS credentials
aws configure

# create our role for EKS
role_arn=$(aws iam create-role --role-name getting-started-eks-role --assume-role-policy-document file://assume-policy.json | jq .Role.Arn | sed s/\"//g)
aws iam attach-role-policy --role-name getting-started-eks-role --policy-arn  arn:aws:iam::aws:policy/AmazonEKSClusterPolicy

# create the cluster VPC

curl https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-05-08/amazon-eks-vpc-sample.yaml -o vpc.yaml
aws cloudformation deploy --template-file vpc.yaml --stack-name getting-started-eks

# grab your stack details 
aws cloudformation list-stack-resources --stack-name getting-started-eks > stack.json

# create our cluster

aws eks create-cluster \
--name getting-started-eks \
--role-arn $role_arn \
--resources-vpc-config subnetIds=subnet-0cf3e8bc8f74e9c90,subnet-0f6ede99566198ed9,subnet-0e58eb56f255dacbf,securityGroupIds=sg-0984913209b1cf929,endpointPublicAccess=true,endpointPrivateAccess=false

aws eks list-clusters
aws eks describe-cluster --name getting-started-eks

aws eks update-kubeconfig --name getting-started-eks --region eu-west-1


#grab the config if you want it
cp ~/.kube/config .

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl

# Install EKS CTL
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
mv /tmp/eksctl /usr/local/bin

yum install -y openssh
mkdir -p ~/.ssh/
PASSPHRASE="1234Qwer"
ssh-keygen -t rsa -b 4096 -N "${PASSPHRASE}" -C "jasoniharris@hotmail.com" -q -f  ~/.ssh/id_rsa
chmod 400 ~/.ssh/id_rsa*



eksctl create cluster --name getting-started-eks \
--region eu-west-1 \
--version 1.16 \
--managed \
--node-type t2.small \
--nodes 1 \
--ssh-access \
--ssh-public-key=~/.ssh/id_rsa.pub \
--node-volume-size 200


# Get kubeconfig for cluster
aws eks update-kubeconfig --name getting-started-eks --region eu-west-1


# Create container
kubectl create ns example-app-2

# lets create some resources.
kubectl apply -n example-app -f secrets/secret.yaml
kubectl apply -n example-app -f configmaps/configmap.yaml
kubectl apply -n example-app -f deployments/deployment.yaml

# remember to change the `type: LoadBalancer`
kubectl apply -n example-app -f services/service.yaml




# lets create some resources.
kubectl apply -n example-app -f secrets/secret.yaml
kubectl apply -n example-app -f configmaps/configmap.yaml
kubectl apply -n example-app -f deployments/deployment.yaml
kubectl apply -n example-app -f services/service.yaml

kubectl apply -n example-app-2 -f secrets/secret.yaml
kubectl apply -n example-app-2 -f configmaps/configmap.yaml
kubectl apply -n example-app-2 -f deployments/deployment2.yaml
kubectl apply -n example-app-2 -f services/service2.yaml

# Useful commands

List cluster


# Handy commands
kubectl get nodes

kubectl get svc -n example-app

kubectl get pods -n example-app


# Clean up
eksctl delete cluster --name getting-started-eks


