# EKS Cluster Setup Guide

This guide covers the installation and configuration of Docker, Jenkins, Trivy, SonarQube, Nexus, AWS CLI, kubectl, and eksctl for an Amazon EKS environment.

---

# Prerequisites

- Create **3 EC2 Instances**
- Instance Type: **t2.medium**
- Storage: **20 GB**
- Ubuntu Operating System

# Install Docker on All 3 VMs

Update the package index:

```bash
sudo apt update
```

## Add Docker's Official GPG Key

```bash
sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

## Add Docker Repository

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update the package index again:

```bash
sudo apt update
```

## Install Docker Packages

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Verify Docker Installation

```bash
docker --version
```

## Grant Permission to Docker Socket (Optional)

```bash
sudo chmod 666 /var/run/docker.sock
```

This command allows Docker commands to be executed without using `sudo`.

**Official Docker Documentation:**

https://docs.docker.com/engine/install/ubuntu/

By following these steps, you should have successfully installed Docker on your Ubuntu system. You can now start using Docker to containerize and manage your applications.


# Jenkins Installation

Follow the official Jenkins installation guide:

https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

## Start and Enable Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## Access Jenkins

Open your browser:

```
http://<Public-IP>:8080
```

Retrieve the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it into the Jenkins setup page.

Then:

- Install Suggested Plugins
- Create the first Admin User

---

# Installing Trivy on Jenkins Server

Link: https://aquasecurity.github.io/trivy/v0.18.3/installation/

 From : Debian/Ubuntu (Official)

---

# SonarQube Setup

SSH into the SonarQube EC2 instance.

Run:

```bash
docker run -d \
--name sonar \
-p 9000:9000 \
sonarqube:lts-community
```

Access SonarQube:

```
http://<Public-IP>:9000
```

Default Credentials

Username:

```
admin
```

Password:

```
admin
```

---

# Nexus Repository Setup

SSH into the Nexus EC2 instance.

Run:

```bash
docker run -d \
--name nexus \
-p 8081:8081 \
sonatype/nexus3
```

Access Nexus:

```
http://<Public-IP>:8081
```

Enter the container:

```bash
docker exec -it <Container-ID> /bin/bash
```

Retrieve the admin password:

```bash
cat /nexus-data/admin.password
```

Default Username:

```
admin
```

> **Note:** Every Nexus installation generates a unique admin password.

---

# EKS Setup

## Step 1: Create an IAM User

Create a new IAM User with any name.

---

## Step 2: Attach the Following Policies

- AmazonEC2FullAccess
- AmazonEKS_CNI_Policy
- AmazonEKSClusterPolicy
- AmazonEKSWorkerNodePolicy
- AWSCloudFormationFullAccess
- IAMFullAccess

---

---

## Step 3: Create a Custom IAM Policy

Create a new IAM policy using the following JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": "eks:*",
      "Resource": "*"
    }
  ]
}
```

Attach this custom policy to the IAM user.

---

# Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

sudo apt update

sudo apt install unzip -y

unzip awscliv2.zip

sudo ./aws/install

aws configure
```

---

# Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

chmod +x kubectl

sudo mv kubectl /usr/local/bin

kubectl version --short --client
```

---

# Install eksctl

```bash
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```

---

# Verification Commands

```bash
docker --version

jenkins --version

aws --version

kubectl version --client

eksctl version
```
# Create EKS Cluster

Create an EKS control plane without a managed node group.

```bash
eksctl create cluster \
  --name=my-eks \
  --region=ap-south-1 \
  --zones=ap-south-1a,ap-south-1b \
  --without-nodegroup
```

---

# Associate IAM OIDC Provider

Associate the IAM OIDC provider with your EKS cluster.

```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster my-eks \
  --approve
```

> **Note:** Ensure the `--region` and `--cluster` values match the cluster you created.

---

# Create a Managed Node Group

```bash
eksctl create nodegroup \
  --cluster=my-eks \
  --region=ap-south-1 \
  --name=node1 \
  --node-type=t3.medium \
  --nodes=3 \
  --nodes-min=2 \
  --nodes-max=4 \
  --node-volume-size=20 \
  --ssh-access \
  --ssh-public-key=<YOUR-KEYPAIR-NAME> \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access
```

> **Note:** Replace `<YOUR-KEYPAIR-NAME>` with the name of your AWS EC2 Key Pair (do **not** include the `.pem` extension).

---

# Verify the Cluster

Check the cluster information:

```bash
eksctl get cluster
```

Check the node group:

```bash
eksctl get nodegroup --cluster my-eks7 --region ap-south-1
```

Verify the worker nodes:

```bash
kubectl get nodes
```

Verify the system pods:

```bash
kubectl get pods -A
```

---

## Security Group Configuration

After the node group is created:

- Open the required **Inbound Rules** in the **Additional Security Group** attached to the worker nodes, if needed for your applications.
  

# Push Project to Your GitHub Repository

Clone the repository and create your own repository, then push the project to your GitHub repository.

## 1. Clone the Repository

```bash
git clone https://github.com/sunil09e/project-Task-Master-Pro.git
```

Change the remote repository:

```bash
git remote set-url origin https://github.com/sunil09e/project-Task-Master-Pro.git
```

> Replace with your GitHub repository.

Or add a new remote:

```bash
git remote add new-origin https://github.com/sunil09e/project-Task-Master-Pro.git
```

> Replace with your GitHub repository.

---

## 2. Initialize Git Repository

```bash
git init
```

---

## 3. Add Files to Git

Stage all files for the first commit:

```bash
git add .
```

---

## 4. Commit Files

Commit the staged files with a commit message:

```bash
git commit -m "Initial commit"
```
```bash
git commit -m "Initial commit"
```

## 5. Push to GitHub

Push the local repository to GitHub:

```bash
git push -u origin main
```

---

# Jenkins Setup Complete

## Install Plugins in Jenkins

1. Eclipse Temurin Installer → for JDK
2. SonarQube Scanner
3. Docker
4. Docker Pipeline
5. Kubernetes
6. Kubernetes CLI
7. Kubernetes Credentials
8. Kubernetes Client API
9. Config File Provider → for Nexus
10. Maven Integration
11. Pipeline Maven Integration

---

Now we have installed the tools. Next, we need to configure them.

Go to:

**Manage Jenkins → Tools**

1. **JDK**
   - Name: `jdk17`
   - Install automatically from Adoptium
   - Version: `JDK 17 (Latest)`

2. **SonarQube Scanner**
   - Name: `sonar-scanner`
   - Install automatically

3. **Maven**
   - Name: `maven3`
   - Version: `3.6.3`
     
4. **Docker**
   - Name: `docker`
   - Install automatically from docker.com

---

Now configure the SonarQube server in Jenkins.

Firstly, generate the token in SonarQube.

Go to:

**Administration → Security → Users → Update Token**

- Name: `sonartoken`
- Click **Generate**



 






