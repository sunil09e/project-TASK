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

# EKS Setup:
# First, create a user in AWS IAM with any name.

# Attach the following policies to the newly created user:
AmazonEC2FullAccess
AmazonEKS_CNI_Policy
AmazonEKSClusterPolicy
AmazonEKSWorkerNodePolicy
AWSCloudFormationFullAccess
IAMFullAccess

# One more policy we need to create with the content below
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






 






