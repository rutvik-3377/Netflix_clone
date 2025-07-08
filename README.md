**Netflix Clone CI/CD with Monitoring | Jenkins | Docker| Kubernetes| Monitoring | DevSecOps**
![intelligence](https://github.com/user-attachments/assets/57ef8f1a-873a-4cf6-9d09-efafa1f28995)


![1_2_Ys33gOVlW3f_MmxI4xKw](https://github.com/user-attachments/assets/b11a528b-08bd-48d1-b762-6b604fc5af2e)

![1_ZAaVlbssxTYlw2pVp7EiAQ](https://github.com/user-attachments/assets/46746216-4d37-447a-b4fb-aaa1b3ff2b42)

![1_jXQbZUEcyI54c0RqAIVh2g](https://github.com/user-attachments/assets/db5b91b3-4834-4316-a4f9-598a1c806076)



Hello friends, we will be deploying a Netflix clone. We will be using Jenkins as a CICD tool and deploying our application on a Docker container and Kubernetes Cluster and we will monitor the Jenkins and Kubernetes metrics using Grafana, Prometheus and Node exporter. I Hope this detailed blog is useful.

**Steps:-**
```
Step 1 — Launch an Ubuntu(22.04) T2 Large Instance

Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.

Step 3 — Create a TMDB API Key.

Step 4 — Install Prometheus and Grafana On the new Server.

Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server.

Step 6 — Email Integration With Jenkins and Plugin setup.

Step 7 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.

Step 8 — Create a Pipeline Project in Jenkins using a Declarative Pipeline

Step 9 — Install OWASP Dependency Check Plugins

Step 10 — Docker Image Build and Push
```

**Contents**  
```
STEP1:Launch an Ubuntu(22.04) T2 Large Instance
Step 2 — Install Jenkins, Docker and Trivy
2A — To Install Jenkins
2B — Install Docker
2C — Install Trivy
Step 3: Create a TMDB API Key
Step 4 — Install Prometheus and Grafana On the new Server
Install Node Exporter on Ubuntu 22.04
Install Grafana on Ubuntu 22.04
Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server
Step 6 — Email Integration With Jenkins and Plugin Setup
Step 7 — Install Plugins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check
7A — Install Plugin
7B — Configure Java and Nodejs in Global Tool Configuration
7C — Create a Job
Step 8 — Configure Sonar Server in Manage Jenkins
Step 9 — Install OWASP Dependency Check Plugins
Step 10 — Docker Image Build and Push
Step 11 — Kuberenetes Setup
Step 11 — Deploy the image using Docker
Step 12 — Kubernetes master and slave setup on Ubuntu (20.04)
Step 13 — Access the Netflix app on the Browser.
Step 14 — Terminate the AWS EC2 Instances.
```

-------------------------------

**STEP1:Launch an Ubuntu(22.04) T2 Large Instance**

Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group and open all ports (not best case to open all ports but just for learning purposes it’s okay).

![image](https://github.com/user-attachments/assets/4b57defc-6dba-47ae-8695-98cd5cac9f67)

**Step 2 — Install Jenkins, Docker and Trivy
2A — To Install Jenkins**




🎥 Netflix-Clone CI/CD DevSecOps Project

This project sets up a DevSecOps pipeline to deploy a Netflix-style application using Jenkins, Docker, Trivy, SonarQube, and a TMDB API key. We also make sure security scans and monitoring integrations are in place.

📅 STEP 1: Launch an Ubuntu (22.04) T2 Large Instance

Go to your AWS Console and launch an EC2 instance.

Choose an Ubuntu 22.04 AMI.

Select an instance type: t2.large.

Create or use an existing key pair.

Configure Security Group:

Open Ports: 22, 80, 443, 8080, 9000, or for learning purposes, open all ports (not recommended in production).

🛠️ STEP 2: Install Jenkins, Docker, Trivy

2A: Install Jenkins

... (same as previous content)

📊 STEP 5: Install Grafana and Monitor Jenkins

... (same as previous content)

📧 STEP 7: Email Integration with Jenkins

... (same as previous content)

🔌 STEP 8: Jenkins Plugins & SonarQube Integration

8A: Install Jenkins Plugins

Go to: Manage Jenkins → Plugins → Available

Install the following plugins without restart:

Eclipse Temurin Installer

SonarQube Scanner

NodeJs Plugin

8B: Configure Global Tool Configuration

Go to: Manage Jenkins → Global Tool Configuration

Add:

JDK 17 (Temurin)

NodeJS 16

SonarScanner

8C: Setup SonarQube Server

Get EC2 Public IP and open http://<ip>:9000

Go to: Administration → Security → Users → Tokens

Create a new token and copy it

In Jenkins:

Go to: Manage Jenkins → Credentials → Global → Add Credentials

Add secret text with your token

Go to: Manage Jenkins → Configure System

Scroll to SonarQube section and add the server:

Name: sonar-server

Server URL: http://<sonarqube-ip>:9000

Token Credential: use the token

8D: Configure Webhook in SonarQube

Go to: Administration → Configuration → Webhooks

Add new webhook:

Name: Jenkins Webhook

URL: http://<jenkins-ip>:8080/sonarqube-webhook/

8E: Jenkins Pipeline Script (with SonarQube)

pipeline {
  agent any
  tools {
    jdk 'jdk17'
    nodejs 'node16'
  }
  environment {
    SCANNER_HOME = tool 'sonar-scanner'
  }
  stages {
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/Aj7Ay/Netflix-clone.git'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar-server') {
          sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
          -Dsonar.projectKey=Netflix '''
        }
      }
    }
    stage('Quality Gate') {
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
        }
      }
    }
    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }
  }
  post {
    always {
      emailext attachLog: true,
        subject: "'${currentBuild.result}'",
        body: "Project: ${env.JOB_NAME}<br/>" +
              "Build Number: ${env.BUILD_NUMBER}<br/>" +
              "URL: ${env.BUILD_URL}<br/>",
        to: 'postbox.aj99@gmail.com',
        attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    }
  }
}

🔐 STEP 9: OWASP & Trivy Scanning

Install OWASP Plugin

Go to: Manage Jenkins → Plugins → Available

Install: OWASP Dependency-Check

Configure OWASP Tool

Go to: Manage Jenkins → Global Tool Configuration

Add new installation for Dependency-Check named DP-Check

Add to Jenkins Pipeline

stage('OWASP FS SCAN') {
  steps {
    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
  }
}
stage('TRIVY FS SCAN') {
  steps {
    sh 'trivy fs . > trivyfs.txt'
  }
}

🐳 STEP 10: Docker Image Build, Scan & Push

Install Docker Plugins in Jenkins

Docker

Docker Commons

Docker Pipeline

Docker API

docker-build-step

Add DockerHub Credentials

Go to: Manage Jenkins → Credentials

Add your DockerHub username & password

ID: docker

Add Docker Stages to Pipeline

stage('Docker Build & Push') {
  steps {
    script {
      withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
        sh 'docker build --build-arg TMDB_V3_API_KEY=Aj7ay86fe14eca3e76869b92 -t netflix .'
        sh 'docker tag netflix sevenajay/netflix:latest'
        sh 'docker push sevenajay/netflix:latest'
      }
    }
  }
}
stage('TRIVY IMAGE SCAN') {
  steps {
    sh 'trivy image sevenajay/netflix:latest > trivyimage.txt'
  }
}
stage('Deploy to container') {
  steps {
    sh 'docker run -d --name netflix -p 8081:80 sevenajay/netflix:latest'
  }
}

Access App at: http://<jenkins-ip>:8081

DockerHub Image: https://hub.docker.com/r/sevenajay/netflix

📅 Next Steps

Setup Jenkins pipeline

Integrate GitHub repo

Add SonarQube, Trivy & Dependency check stages

Push Docker image

Deploy to Kubernetes

Setup Prometheus, Node Exporter & Grafana

Configure Email notifications

Continue with full CI/CD + Monitoring + DevSecOps flow...

