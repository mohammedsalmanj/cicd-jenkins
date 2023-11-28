# Jenkins Installation Steps

# Check if Jenkins is running on port 8080 after installation
check port 8080

# Retrieve initial admin password for Jenkins
cat /var/lib/jenkins/secrets/initialAdminPassword

# Install and configure Kops, create a Kubernetes cluster

# Install kubectl
curl -LO "https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install KOPS
wget https://github.com/kubernetes/kops/releases/download/v1.26.4/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

# Create and manage a Kubernetes cluster
kops create cluster --name=kube.dealz4u.app --state=s3://kube-buc --zones=us-east-1a,us-east-1b --node-count=2 --node-size=t3.small --master-size=t3.small --dns-zone=kube.dealz4u.app --node-volume-size=8 --master-volume-size=8
kops update cluster --name kube.dealz4u.app --state=s3://kube-buc --yes --admin
kops validate cluster --name kube.dealz4u.app --state=s3://kube-buc --yes

# Display Kubernetes configuration
cat ~/.kube/config

# List nodes in the Kubernetes cluster
kubectl get nodes

# Install Docker
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
sudo service docker start

# Add Jenkins user to Docker group
usermod -aG docker jenkins
# Reboot Jenkins and connect again

# Start Kubernetes nodes and create a cluster

# Install Helm in Kops server
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# Move Helm binary to /usr/local/bin/
sudo mv /usr/bin/helm /usr/local/bin/

# Verify Helm installation
which helm

# Add Git repository to server and clone source code
# Copy all to a new repository
cp -r * ../cicd-jenkins/
cd cicd-jenkins
rm -rf Docker-db Docker-web compose ansible Docker-app helm

# Create Dockerfile for Tomcat
vim Dockerfile
FROM tomcat:8-jre11
RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]

# Create Helm chart
cd helm/
helm create vprofilecharts
# Copy Kubernetes YAML files to Helm templates
cp kubernetes/vpro-app/* helm/vprofilecharts/templates

# Update Helm deployment YAML with the Docker image
# Run Helm deployment
helm install --namespace test vprofile-stack helm/vprofilecharts/ --set appimage=imranvisualpath/vproappdock:9

# Check AWS configuration and save to file
aws configure list | jq -r 'to_entries | map("\(.key)=\(.value|tostring)") | join("\n")' > aws-config.yaml

# Verify Kubernetes deployment
kubectl get all --namespace test

# Delete Helm deployment
helm delete vprofile-stack --namespace test

# Generate SSH key for Git authentication
ssh-keygen
cat ~/.ssh/id_rsa.pub
git remote set-url origin git@github.com:mohammedsalmanj/cicd-jenkins.git

# Jenkins slave setup

# Install Java and create a directory for Jenkins slave
sudo apt install openjdk-8-jdk -y
sudo mkdir /opt/jenkins-slave
sudo chown ubuntu.ubuntu /opt/jenkins-slave/ -R
java --version

# Update security group to allow Jenkins to login to Kops server

# Add SonarScanner as a tool named "scanner4"

# Check Kubernetes cluster status
kubectl get all --namespace prod
helm list --namespace prod
kubectl get pods --namespace prod
kubectl describe pods --namespace prod
kubectl get svc --namespace prod
