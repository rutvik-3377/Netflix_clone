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
Connect to your console, and enter these commands to Install Jenkins

```
vi jenkins.sh #make sure run in Root (or) add at userdata while ec2 launch
```
```
#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```
```
sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins
```
Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.


Now, grab your Public IP Address
```
<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/35c64c29-84d8-482a-8601-21853fd49a43" />

Unlock Jenkins using an administrative password and install the suggested plugins.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/55565c01-a266-4295-af10-78bd981de059" />

Jenkins will now get installed and install all the libraries.

Create a user click on save and continue.

Jenkins Getting Started Screen.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/37fd5e58-8c31-44f3-9f86-470fc282eb74" />

**2B — Install Docker**

```
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

After the docker installation, we create a sonarqube container (Remember to add 9000 ports in the security group).

```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
<img width="1772" height="370" alt="image" src="https://github.com/user-attachments/assets/69c0b078-616f-40c5-9264-3ff57a864d05" />

Now our sonarqube is up and running

<img width="1077" height="517" alt="image" src="https://github.com/user-attachments/assets/1b4ac706-569a-40e5-b0ec-fb12da9f4e7a" />

Enter username and password, click on login and change password.

```
username admin
password admin
```
Update New password, This is Sonar Dashboard.


<img width="1920" height="616" alt="image" src="https://github.com/user-attachments/assets/7dea8cda-ca53-4881-96c1-11ad7eebb94e" />

**2C — Install Trivy**

```
vi trivy.sh
```
```
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

**Step 3: Create a TMDB API Key**

Next, we will create a TMDB API key

Open a new tab in the Browser and search for TMDB

<img width="1905" height="751" alt="image" src="https://github.com/user-attachments/assets/62727d91-3f35-4eb3-8d4a-f74322e27702" />


Click on the first result, you will see this page

<img width="1903" height="898" alt="image" src="https://github.com/user-attachments/assets/b9b5a077-7e74-4690-a6a9-a61370430343" />


Click on the Login on the top right. You will get this page.

You need to create an account here. click on click here. I have account that’s why i added my details there.

<img width="1873" height="557" alt="image" src="https://github.com/user-attachments/assets/465afd88-639a-4242-9ad2-afcf6682561a" />


once you create an account you will see this page.


<img width="1906" height="534" alt="image" src="https://github.com/user-attachments/assets/530c1d99-0fc6-419a-bde7-12c9b34202e5" />

Let’s create an API key, By clicking on your profile and clicking settings.

<img width="1803" height="861" alt="image" src="https://github.com/user-attachments/assets/20ef8b18-916a-4c67-88da-4a24e081a560" />


Now click on API from the left side panel.

<img width="1713" height="861" alt="image" src="https://github.com/user-attachments/assets/081b3661-70d2-48c9-92bd-916bc78346ed" />

<img width="1839" height="695" alt="image" src="https://github.com/user-attachments/assets/c5a72d4d-5ef4-476d-9496-e1d0c185d7e2" />


Click on Developer

<img width="1729" height="503" alt="image" src="https://github.com/user-attachments/assets/59bbab6d-27cf-41c3-bcbf-9f3c58b5f78c" />


Now you have to accept the terms and conditions.


<img width="1455" height="867" alt="image" src="https://github.com/user-attachments/assets/f57f1697-5895-4b84-9022-a16729ab26c5" />

Provide basic details.

<img width="1887" height="635" alt="image" src="https://github.com/user-attachments/assets/851d31ef-9c68-465e-ba14-c8e18f7c0edd" />


<img width="1835" height="929" alt="image" src="https://github.com/user-attachments/assets/e1eedb2a-2896-4505-8afa-46a6b1c077aa" />


Click on submit and you will get your API key.

<img width="1829" height="655" alt="image" src="https://github.com/user-attachments/assets/e637eb43-d218-46a3-b9ba-0672c8fd5775" />


**Step 4 — Install Prometheus and Grafana On the new Server**

First of all, let’s create a dedicated Linux user sometimes called a system account for Prometheus. Having individual users for each service serves two main purposes:

It is a security measure to reduce the impact in case of an incident with the service.

It simplifies administration as it becomes easier to track down what resources belong to which service.

To create a system user or system account, run the following command:

```
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false prometheus
```

<img width="473" height="157" alt="image" src="https://github.com/user-attachments/assets/4e01cce1-dce7-4ffe-b191-d43247fa854a" />


–system – Will create a system account.
–no-create-home – We don’t need a home directory for Prometheus or any other system accounts in our case.
–shell /bin/false – It prevents logging in as a Prometheus user.
Prometheus – Will create a Prometheus user and a group with the same name.

Let’s check the latest version of Prometheus from the download page.

You can use the curl or wget command to download Prometheus.

```
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```

<img width="1819" height="593" alt="image" src="https://github.com/user-attachments/assets/14a94d3c-4265-4a7c-bfb4-0fc363137d8e" />

Then, we need to extract all Prometheus files from the archive.

```
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
```

<img width="813" height="452" alt="image" src="https://github.com/user-attachments/assets/79171916-7cf6-4de0-84e7-71fe1fbdc7cf" />


Usually, you would have a disk mounted to the data directory. For this tutorial, I will simply create a /data directory. Also, you need a folder for Prometheus configuration files.


```
sudo mkdir -p /data /etc/prometheus
```

<img width="704" height="82" alt="image" src="https://github.com/user-attachments/assets/a480dcb0-1b31-4e7b-bdc8-523a1ca3f194" />


Now, let’s change the directory to Prometheus and move some files.


```
cd prometheus-2.47.1.linux-amd64/
```


<img width="782" height="305" alt="image" src="https://github.com/user-attachments/assets/2bbe777c-440a-4fda-ada9-d59f3991425a" />

First of all, let’s move the Prometheus binary and a promtool to the /usr/local/bin/. promtool is used to check configuration files and Prometheus rules.


```
sudo mv prometheus promtool /usr/local/bin/
```


<img width="1170" height="118" alt="image" src="https://github.com/user-attachments/assets/d5899f5c-ea46-4ae6-b799-a60c6689c4cd" />


Optionally, we can move console libraries to the Prometheus configuration directory. Console templates allow for the creation of arbitrary consoles using the Go templating language. You don’t need to worry about it if you’re just getting started.


```
sudo mv consoles/ console_libraries/ /etc/prometheus/
```


<img width="1127" height="121" alt="image" src="https://github.com/user-attachments/assets/4e6932ca-fd7c-4d4c-8152-2ddbd6e0b0ee" />


Finally, let’s move the example of the main Prometheus configuration file.

```
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

To avoid permission issues, you need to set the correct ownership for the /etc/prometheus/ and data directory.

```
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

You can delete the archive and a Prometheus folder when you are done.


```
cd
rm -rf prometheus-2.47.1.linux-amd64.tar.gz
```


<img width="1254" height="483" alt="image" src="https://github.com/user-attachments/assets/cf266b76-8120-4647-930a-465cb3f1cd8f" />


Verify that you can execute the Prometheus binary by running the following command:


```
prometheus --version
```


<img width="1027" height="214" alt="image" src="https://github.com/user-attachments/assets/e094b638-f777-4d57-ae8d-459b5c1a5ef7" />

To get more information and configuration options, run Prometheus Help.


```
prometheus --help
```

We’re going to use some of these options in the service definition.

We’re going to use Systemd, which is a system and service manager for Linux operating systems. For that, we need to create a Systemd unit configuration file.

```
sudo vim /etc/systemd/system/prometheus.service
```


<img width="797" height="81" alt="image" src="https://github.com/user-attachments/assets/b4217c4d-d096-447c-a1a8-da942f908a3e" />

Prometheus.service

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle
[Install]
WantedBy=multi-user.target

```


<img width="899" height="497" alt="image" src="https://github.com/user-attachments/assets/441a7d24-04d1-400a-a9e6-4abc40f5486a" />


Let’s go over a few of the most important options related to Systemd and Prometheus. Restart – Configures whether the service shall be restarted when the service process exits, is killed, or a timeout is reached.
RestartSec – Configures the time to sleep before restarting a service.
User and Group – Are Linux user and a group to start a Prometheus process.
–config.file=/etc/prometheus/prometheus.yml – Path to the main Prometheus configuration file.
–storage.tsdb.path=/data – Location to store Prometheus data.
–web.listen-address=0.0.0.0:9090 – Configure to listen on all network interfaces. In some situations, you may have a proxy such as nginx to redirect requests to Prometheus. In that case, you would configure Prometheus to listen only on localhost.
–web.enable-lifecycle — Allows to manage Prometheus, for example, to reload configuration without restarting the service.

To automatically start the Prometheus after reboot, run enable.


```
sudo systemctl enable prometheus
```


<img width="1243" height="120" alt="image" src="https://github.com/user-attachments/assets/c91b1f0f-ca74-4324-ab6c-0d4ed6cfb443" />


Then just start the Prometheus.


```
sudo systemctl start prometheus
```


<img width="808" height="100" alt="image" src="https://github.com/user-attachments/assets/41f43064-e122-48e8-a43e-25788d030798" />


To check the status of Prometheus run the following command:


```
sudo systemctl status prometheus
```

<img width="1841" height="484" alt="image" src="https://github.com/user-attachments/assets/d32c575c-6ed2-40a5-8877-db83e08b7e26" />


Suppose you encounter any issues with Prometheus or are unable to start it. The easiest way to find the problem is to use the journalctl command and search for errors.


```
journalctl -u prometheus -f --no-pager
```

Now we can try to access it via the browser. I’m going to be using the IP address of the Ubuntu server. You need to append port 9090 to the IP.

```
<public-ip:9090>
```


<img width="1906" height="603" alt="image" src="https://github.com/user-attachments/assets/33dd7ea3-efbe-438a-b6d6-4e06e5e5a612" />

If you go to targets, you should see only one – Prometheus target. It scrapes itself every 15 seconds by default.


**Install Node Exporter on Ubuntu 22.04**


Next, we’re going to set up and configure Node Exporter to collect Linux system metrics like CPU load and disk I/O. Node Exporter will expose these as Prometheus-style metrics. Since the installation process is very similar, I’m not going to cover as deep as Prometheus.

First, let’s create a system user for Node Exporter by running the following command:

```
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter
```


<img width="1220" height="144" alt="image" src="https://github.com/user-attachments/assets/020a8a48-bdb5-4242-8dbb-025999fe5687" />


You can download Node Exporter from the same page.

Use the wget command to download the binary.


```
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```



<img width="1852" height="537" alt="image" src="https://github.com/user-attachments/assets/e22f3891-4488-4a5d-938b-2337c381e778" />


Extract the node exporter from the archive.


```
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
```


<img width="1129" height="251" alt="image" src="https://github.com/user-attachments/assets/2240d33c-c6bf-402f-bdfb-a97ebe0ecc9e" />


Move binary to the /usr/local/bin.


```
sudo mv \
  node_exporter-1.6.1.linux-amd64/node_exporter \
  /usr/local/bin/
```


<img width="519" height="145" alt="image" src="https://github.com/user-attachments/assets/6b2f5ac6-7926-451a-bd95-faa650d0cb7e" />


Clean up, and delete node_exporter archive and a folder.


```
rm -rf node_exporter*
```


<img width="1068" height="244" alt="image" src="https://github.com/user-attachments/assets/7d1215fe-2229-4e4d-8105-ad406c466aa3" />


Verify that you can run the binary.


```
node_exporter --version
```

Node Exporter has a lot of plugins that we can enable. If you run Node Exporter help you will get all the options.


```
node_exporter --help
```

–collector.logind We’re going to enable the login controller, just for the demo.

Next, create a similar systemd unit file.


```
sudo vim /etc/systemd/system/node_exporter.service
```

**node_exporter.service**

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind
[Install]
WantedBy=multi-user.target
```


<img width="1121" height="406" alt="image" src="https://github.com/user-attachments/assets/2b983c3a-78df-4f08-a872-62118595cd25" />


Replace Prometheus user and group to node_exporter, and update the ExecStart command.

To automatically start the Node Exporter after reboot, enable the service.


```
sudo systemctl enable node_exporter
```

Then start the Node Exporter.


```
sudo systemctl start node_exporter
```

Check the status of Node Exporter with the following command:


```
sudo systemctl status node_exporter
```

If you have any issues, check logs with journalctl

```
journalctl -u node_exporter -f --no-pager
```

At this point, we have only a single target in our Prometheus. There are many different service discovery mechanisms built into Prometheus. For example, Prometheus can dynamically discover targets in AWS, GCP, and other clouds based on the labels. In the following tutorials, I’ll give you a few examples of deploying Prometheus in a cloud-specific environment. For this tutorial, let’s keep it simple and keep adding static targets. Also, I have a lesson on how to deploy and manage Prometheus in the Kubernetes cluster.

To create a static target, you need to add job_name with static_configs.


```
sudo vim /etc/prometheus/prometheus.yml
```


<img width="929" height="103" alt="image" src="https://github.com/user-attachments/assets/d63554ac-7917-4956-bc36-0990e637fbdb" />

**prometheus.yml**

```
- job_name: node_export
    static_configs:
      - targets: ["localhost:9100"]
```


<img width="1083" height="739" alt="image" src="https://github.com/user-attachments/assets/14bde08b-a501-494d-a21e-5d8c9b145b95" />

By default, Node Exporter will be exposed on port 9100.

Since we enabled lifecycle management via API calls, we can reload the Prometheus config without restarting the service and causing downtime.

Before, restarting check if the config is valid.


```
promtool check config /etc/prometheus/prometheus.yml
```


<img width="1189" height="140" alt="image" src="https://github.com/user-attachments/assets/2501dc7a-cc98-4b61-84eb-00004274b7aa" />


Then, you can use a POST request to reload the config.


```
curl -X POST http://localhost:9090/-/reload
```


<img width="951" height="92" alt="image" src="https://github.com/user-attachments/assets/e52857ca-afbb-4599-936c-3cb62dff6b32" />

Check the targets section

```
http://<ip>:9090/targets
```


<img width="1920" height="703" alt="image" src="https://github.com/user-attachments/assets/b11b823e-867d-44be-b54c-27a0105eaa08" />


**Install Grafana on Ubuntu 22.04**
