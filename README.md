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

To visualize metrics we can use Grafana. There are many different data sources that Grafana supports, one of them is Prometheus.

First, let’s make sure that all the dependencies are installed.

```
sudo apt-get install -y apt-transport-https software-properties-common
```
<img width="1368" height="792" alt="image" src="https://github.com/user-attachments/assets/d0dc6218-f9e9-43af-8fca-25cb847bf2fc" />

Next, add the GPG key.

```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

<img width="1085" height="120" alt="image" src="https://github.com/user-attachments/assets/4514559c-cac5-4e40-ae5d-f7f508aed4ff" />


Add this repository for stable releases.

```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
<img width="1444" height="157" alt="image" src="https://github.com/user-attachments/assets/bfaa4a54-e30d-4135-a81c-6302ee38cc99" />


After you add the repository, update and install Garafana.

```
sudo apt-get update
```

```
sudo apt-get -y install grafana
```

<img width="1405" height="386" alt="image" src="https://github.com/user-attachments/assets/8f55d13d-9ba6-4214-a73e-593a4026e403" />


To automatically start the Grafana after reboot, enable the service.

```
sudo systemctl enable grafana-server
```

Then start the Grafana.


```
sudo systemctl start grafana-server
```

To check the status of Grafana, run the following command:


```
sudo systemctl status grafana-server
```

<img width="1850" height="494" alt="image" src="https://github.com/user-attachments/assets/0e2783aa-b216-4ecd-aab0-ed2b4687f448" />


Go to http://<ip>:3000 and log in to the Grafana using default credentials. The username is admin, and the password is admin as well.


```
username admin
password admin
```

<img width="1914" height="896" alt="image" src="https://github.com/user-attachments/assets/05bf10c5-bcea-4e53-852b-bc976875b93b" />


When you log in for the first time, you get the option to change the password.

To visualize metrics, you need to add a data source first.


<img width="1917" height="785" alt="image" src="https://github.com/user-attachments/assets/8d4b1844-b0d8-4954-b0f6-f74c643d6e27" />


Click Add data source and select Prometheus.

<img width="1911" height="851" alt="image" src="https://github.com/user-attachments/assets/94aec513-9cec-4782-9c31-8c4e1cb55a91" />


For the URL, enter localhost:9090 and click Save and test. You can see Data source is working.


```
<public-ip:9090>
```
<img width="1917" height="921" alt="image" src="https://github.com/user-attachments/assets/a883ad15-4c53-4652-b216-dcaa2fa697db" />


Click on Save and Test.


<img width="1919" height="999" alt="image" src="https://github.com/user-attachments/assets/0928a029-949c-4989-9b9d-e7ecd702a55c" />


Let’s add Dashboard for a better view

<img width="1899" height="715" alt="image" src="https://github.com/user-attachments/assets/8bf24289-9c6d-4cd4-9ba9-b1dfc845cb63" />


Click on Import Dashboard paste this code 1860 and click on load

<img width="1851" height="547" alt="image" src="https://github.com/user-attachments/assets/146346e0-e71a-4bf1-ad87-0d31a0d8807a" />


Select the Datasource and click on Import

<img width="1919" height="991" alt="image" src="https://github.com/user-attachments/assets/e8eaa3df-e1a0-4179-89d2-f73da9a14169" />


You will see this output

<img width="1909" height="889" alt="image" src="https://github.com/user-attachments/assets/adcfa109-2f12-4945-8145-fca8c192c290" />


**Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server**


Let’s Monitor JENKINS SYSTEM

Need Jenkins up and running machine

Goto Manage Jenkins –> Plugins –> Available Plugins

Search for Prometheus and install it

<img width="1897" height="559" alt="image" src="https://github.com/user-attachments/assets/38001152-e79f-487b-a633-e53828d85aaa" />


Once that is done you will Prometheus is set to /Prometheus path in system configurations

<img width="1825" height="899" alt="image" src="https://github.com/user-attachments/assets/2fa6b0e0-9970-4b34-a0d0-76476b034f69" />


Nothing to change click on apply and save

To create a static target, you need to add job_name with static_configs. go to Prometheus server


```
sudo vim /etc/prometheus/prometheus.yml
```

<img width="689" height="96" alt="image" src="https://github.com/user-attachments/assets/3ee9cb78-10f2-41d7-8595-d225dc718963" />


Paste below code

```
- job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<jenkins-ip>:8080']
```

<img width="1005" height="787" alt="image" src="https://github.com/user-attachments/assets/6da4582b-88c9-49ed-98bd-f8cb39cbb3d5" />


Before, restarting check if the config is valid.

```
promtool check config /etc/prometheus/prometheus.yml
```

Then, you can use a POST request to reload the config.

```
curl -X POST http://localhost:9090/-/reload
```

<img width="819" height="170" alt="image" src="https://github.com/user-attachments/assets/39e8b8ad-0df9-49e0-9ca1-d3c87879eb79" />


Check the targets section

```
http://<ip>:9090/targets
```

You will see Jenkins is added to it


<img width="1907" height="879" alt="image" src="https://github.com/user-attachments/assets/321b524d-c2b7-43d4-8eab-c1fb9bbdc220" />


Let’s add Dashboard for a better view in Grafana

Click On Dashboard –> + symbol –> Import Dashboard

Use Id 9964 and click on load

<img width="1917" height="643" alt="image" src="https://github.com/user-attachments/assets/66e49403-2443-48ce-b611-d07ce5290618" />


Select the data source and click on Import

<img width="1917" height="991" alt="image" src="https://github.com/user-attachments/assets/545d0504-2a1e-4125-983b-32781ed18c26" />


Now you will see the Detailed overview of Jenkins

<img width="1913" height="993" alt="image" src="https://github.com/user-attachments/assets/e1260f3d-3a3b-4ac9-b8bd-1ec9b5145c2f" />


**Step 6 — Email Integration With Jenkins and Plugin Setup**

Install Email Extension Plugin in Jenkins


<img width="1847" height="545" alt="image" src="https://github.com/user-attachments/assets/b8de95c2-d221-43b2-9d0d-aaf2fcfd774d" />

Go to your Gmail and click on your profile

Then click on Manage Your Google Account –> click on the security tab on the left side panel you will get this page(provide mail password).

<img width="1873" height="733" alt="image" src="https://github.com/user-attachments/assets/34cc4ef9-a9d6-4889-a575-6714e95791fa" />


2-step verification should be enabled.

Search for the app in the search bar you will get app passwords like the below image.

<img width="1507" height="517" alt="image" src="https://github.com/user-attachments/assets/6b47c2b4-8f5f-436d-a83e-26b4015c11d0" />

<img width="1710" height="549" alt="image" src="https://github.com/user-attachments/assets/8656b5b5-2d0b-4629-a585-d6242a4b18b6" />


Click on other and provide your name and click on Generate and copy the password

<img width="817" height="657" alt="image" src="https://github.com/user-attachments/assets/6487107b-59cc-462a-a3a9-dc2385c99f32" />


In the new update, you will get a password like this

<img width="1141" height="751" alt="image" src="https://github.com/user-attachments/assets/81d425f6-4583-4945-9c77-688ef9c9983e" />


Once the plugin is installed in Jenkins, click on manage Jenkins –> configure system there under the E-mail Notification section configure the details as shown in the below image.

<img width="1851" height="827" alt="image" src="https://github.com/user-attachments/assets/c6052210-2984-40d7-91ff-0cd8a3262ce1" />

<img width="1605" height="875" alt="image" src="https://github.com/user-attachments/assets/7bad8eed-1138-48e5-8031-25ec000d5983" />


Click on Apply and save.

Click on Manage Jenkins–> credentials and add your mail username and generated password.

<img width="1819" height="879" alt="image" src="https://github.com/user-attachments/assets/39a50f3c-d54b-4c5a-9d8e-ffaa395fdb76" />


This is to just verify the mail configuration

Now under the Extended E-mail Notification section configure the details as shown in the below images.

<img width="1837" height="747" alt="image" src="https://github.com/user-attachments/assets/fdfe24a3-4475-4b95-ac27-34d879a9c0a6" />

<img width="1851" height="783" alt="image" src="https://github.com/user-attachments/assets/da09a630-c691-41fa-9d9a-6009ae566428" />

<img width="1761" height="881" alt="image" src="https://github.com/user-attachments/assets/145b98bf-7068-4385-8acb-555c92090adf" />


Click on Apply and save.

```
post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'postbox.aj99@gmail.com',  #change Your mail
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
```

Next, we will log in to Jenkins and start to configure our Pipeline in Jenkins


**Step 7 — Install Plugins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check**

**7A — Install Plugin**

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 → Eclipse Temurin Installer (Install without restart)

2 → SonarQube Scanner (Install without restart)

3 → NodeJs Plugin (Install Without restart)

<img width="1920" height="749" alt="image" src="https://github.com/user-attachments/assets/0a85e392-c755-4d12-aa91-50422d5556a2" />


**7B — Configure Java and Nodejs in Global Tool Configuration**

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

<img width="1909" height="634" alt="image" src="https://github.com/user-attachments/assets/63344277-3d9f-4168-8f91-9086c28c3b19" />


<img width="1693" height="719" alt="image" src="https://github.com/user-attachments/assets/fb528800-c40e-4c3a-b082-b55a7b29d926" />


**7C — Create a Job**

create a job as Netflix Name, select pipeline and click on ok.

**Step 8 — Configure Sonar Server in Manage Jenkins**

Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token


<img width="1107" height="299" alt="image" src="https://github.com/user-attachments/assets/fd68342e-4ce5-4f23-a7bf-afa1db3a04fd" />

click on update Token

<img width="1610" height="199" alt="image" src="https://github.com/user-attachments/assets/ea892a39-dfb0-44f0-a16a-5d52190ec53c" />


Create a token with a name and generate

<img width="1568" height="444" alt="image" src="https://github.com/user-attachments/assets/fc4eed78-2135-48cb-a678-175ec914b989" />


copy Token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

<img width="1805" height="813" alt="image" src="https://github.com/user-attachments/assets/f337a826-dd65-4249-96b9-5170ea23f623" />


You will this page once you click on create

<img width="1633" height="202" alt="image" src="https://github.com/user-attachments/assets/4f026747-ab0f-4f84-a48f-fe89bd7b77f4" />


Now, go to Dashboard → Manage Jenkins → System and Add like the below image.

<img width="1747" height="883" alt="image" src="https://github.com/user-attachments/assets/29376bfa-5350-4ed4-b4f3-57656c8ded1f" />


Click on Apply and Save

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

<img width="1721" height="893" alt="image" src="https://github.com/user-attachments/assets/6c0af23d-8c75-4c98-9304-f78114ce1eb8" />


In the Sonarqube Dashboard add a quality gate also

Administration–> Configuration–>Webhooks

<img width="1757" height="795" alt="image" src="https://github.com/user-attachments/assets/12df102d-e5e0-4e05-bd40-9bd3caea91c6" />

Click on Create

<img width="1701" height="374" alt="image" src="https://github.com/user-attachments/assets/b32ca7f0-40fb-4f6e-9ca3-39d4a94b010e" />

Add details

```
#in url section of quality gate
<http://jenkins-public-ip:9090>/sonarqube-webhook/
```

<img width="1857" height="941" alt="image" src="https://github.com/user-attachments/assets/26f60c8e-bddb-45a1-a23d-b43d59f5599d" />

Let’s go to our Pipeline and add the script in our Pipeline Script.

```
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Aj7Ay/Netflix-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
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
            to: 'rutvik@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
```

Click on Build now, you will see the stage view like this

<img width="846" height="268" alt="image" src="https://github.com/user-attachments/assets/4a5c813e-4b06-4dd5-8abe-09d56e79e850" />

To see the report, you can go to Sonarqube Server and go to Projects.

<img width="1247" height="197" alt="image" src="https://github.com/user-attachments/assets/8bbe35d7-e8d6-4def-adf3-b9684112e72a" />

You can see the report has been generated and the status shows as passed. You can see that there are 3.2k lines it scanned. To see a detailed report, you can go to issues.

** Step 9 — Install OWASP Dependency Check Plugins**

GotoDashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

<img width="1853" height="451" alt="image" src="https://github.com/user-attachments/assets/9a856b59-d178-476f-8e6a-b87cfc4bf0d8" />

First, we configured the Plugin and next, we had to configure the Tool

Goto Dashboard → Manage Jenkins → Tools →

<img width="1015" height="691" alt="image" src="https://github.com/user-attachments/assets/93286acb-0e9d-4b42-ade5-50e96d002870" />

Click on Apply and Save here.

Now go configure → Pipeline and add this stage to your pipeline and build.


```
stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

```

The stage view would look like this,

<img width="1106" height="263" alt="image" src="https://github.com/user-attachments/assets/9eead550-f68e-4a39-ad52-250066097fc3" />


You will see that in status, a graph will also be generated and Vulnerabilities.

<img width="1435" height="711" alt="image" src="https://github.com/user-attachments/assets/9b28e0c2-3b02-42e5-a5fa-99a05b480cf7" />


**Step 10 — Docker Image Build and Push**

We need to install the Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins.

  -  Docker

  - Docker Commons

  - Docker Pipeline

  - Docker API

  - docker-build-step

and click on install without restart

<img width="1837" height="873" alt="image" src="https://github.com/user-attachments/assets/ba5fba4a-383a-4d18-866a-d350ba658615" />

Now, goto Dashboard → Manage Jenkins → Tools →

<img width="1833" height="711" alt="image" src="https://github.com/user-attachments/assets/045b1a4c-4ebf-43da-b0fe-65af7df2b8d6" />


Add DockerHub Username and Password under Global Credentials

<img width="1791" height="895" alt="image" src="https://github.com/user-attachments/assets/7e08b1ac-fd90-4793-845d-00e828631b3d" />


Add this stage to Pipeline Script

```
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build --build-arg TMDB_V3_API_KEY=Aj7ay86fe14eca3e76869b92 -t netflix ."
                       sh "docker tag netflix sevenajay/netflix:latest "
                       sh "docker push sevenajay/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sevenajay/netflix:latest > trivyimage.txt"
            }
        }
```

You will see the output below, with a dependency trend.

<img width="1146" height="678" alt="image" src="https://github.com/user-attachments/assets/2021ada6-54f2-4a34-8c37-a311d781964a" />

<img width="1146" height="678" alt="image" src="https://github.com/user-attachments/assets/2e70d091-c4d2-45be-ac05-0d533c566feb" />


When you log in to Dockerhub, you will see a new image is created

<img width="1540" height="172" alt="image" src="https://github.com/user-attachments/assets/43a4fdf2-02c4-4f2a-82b2-c7f9670b0257" />

Now Run the container to see if the game coming up or not by adding the below stage

```
stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 sevenajay/netflix:latest'
            }
        }
```

stage view

<img width="1272" height="304" alt="image" src="https://github.com/user-attachments/assets/6d168c1f-dd3e-4ce9-8c35-42392bc6b01b" />

```
<Jenkins-public-ip:8081>
```

You will get this output

<img width="1881" height="995" alt="image" src="https://github.com/user-attachments/assets/d3917b25-47b7-4c3c-9ea2-4bac5f6bc6aa" />


