Kubernetes Cluster on AWS
Kirk Kalvar
Updated 10/20/2016

# provision amazon-linux-ami 
Amazon Linux AMI 2016.03.3 (HVM), SSD Volume Type - ami-6869aa05, Free tier eligible t2.micro, Free tier eligible which has the awscli tools pre-installed

# login into k8s-console  using ssh and configure awscli 
aws configure
AWS Access Key ID [****************SSKA]:
AWS Secret Access Key [****************IOxz]:
Default region name [us-east-1]:
Default output format [None]:

# add docker 
sudo yum install -y docker git
sudo service docker start
sudo usermod -a -G docker ec2-user

# test docker
sudo docker images
sudo docker run hello-world

# Dockerfile and kubernetes fixes
git clone https://github.com/kskalvar/kubernetes-aws.git

# install kubernetes v1.4.3
curl -sS https://get.k8s.io | sed '$d' | bash
 
# set kubernetes environment variables
export NUM_NODES=3
export KUBE_AWS_INSTANCE_PREFIX=k8s
export KUBE_AWS_ZONE=us-east-1a
export MASTER_SIZE=t2.micro
export AWS_S3_REGION=us-east-1
export NODE_SIZE=t2.micro
export KUBERNETES_PROVIDER=aws

# start cluster (takes about 10 minutes)
kubernetes/cluster/kube-up.sh

# test kubernetes
export PATH=/home/ec2-user/kubernetes/platforms/linux/amd64:$PATH
kubectl get nodes

# create container
cd kubernetes-aws/web
sudo docker build -t kskalvar/web  .
sudo docker login
sudo docker push kskalvar/web

# create pod
kubectl run web --image=kskalvar/web --port=5000

# scale pod
kubectl scale deployment web --replicas=2

# show pods running
kubectl get pods --output wide

# create load balancer
kubectl expose deployment web --port=80 --target-port=5000 --type="LoadBalancer"

# get aws external load balancer external address
kubectl get service web --output wide

# test from browser

# kill application
kubectl delete deployment,service web

# shutdown cluster
kubernetes/cluster/kube-down.sh
