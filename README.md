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


