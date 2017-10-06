Kubernetes Cluster on AWS
Kirk Kalvar
Updated 09/27/2017

# provision amazon-linux-ami 
Amazon Linux AMI 2016.03.3 (HVM), SSD Volume Type - ami-6869aa05, Free tier eligible t2.micro,
Free tier eligible which has the awscli tools pre-installed

# login into amazon-linux-ami using ssh and configure aws client tools
aws configure
AWS Access Key ID [****************SSKA]:
AWS Secret Access Key [****************IOxz]:
Default region name [us-east-1]:
Default output format [None]:

# add docker 
sudo yum install -y docker git
sudo service docker start
sudo usermod -a -G docker ec2-user

# test docker (you may need to logout/login)
docker images
docker run hello-world

# get Dockerfile
git clone https://github.com/kskalvar/kubernetes-aws.git

# install kubernetes v1.7.6
export KUBERNETES_RELEASE=v1.7.6
export KUBERNETES_PROVIDER=aws
export KUBERNETES_SKIP_CONFIRM=true

curl -sS https://get.k8s.io | sed '$d' | bash

# test kubectl
export PATH=/home/ec2-user/kubernetes/platforms/linux/amd64:$PATH
kubectl

# install kops v1.7.0
wget http://github.com/kubernetes/kops/releases/download/1.7.0/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

# test kops
export PATH=/usr/local/bin:$PATH
kops

# generate public/private rsa key pair
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa

# cleanup
rm -f kubernetes.tar.gz

# create route53 subdomain
aws route53 create-hosted-zone --name dev.kal.technology --caller-reference {1,2,3 ... n: where n is different everytime}

# Add subdomain NS record to domain (please see kubernetes installation instructions)

# test route53 (should see 4 NS records if working correctly in ANSWER SECTION)
dig NS dev.kal.technology

# add aws s3 storage
aws s3 mb s3://kube.dev.kal.technology
export KOPS_STATE_STORE=s3://kube.dev.kal.technology

# configure cluster
kops create cluster --zones=us-east-1a ksk.useast1.kube.dev.kal.technology

# create cluster
kops update cluster ksk.useast1a.kube.dev.kal.technology --yes

# validate cluster
kops validate cluster

# create container
cd kubernetes-aws/web
docker build -t kskalvar/web  .

# login to docker hub and push container
docker login
docker push kskalvar/web

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
kops delete cluster ksk.useast1a.kube.dev.kal.technology --yes
