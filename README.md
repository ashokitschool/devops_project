
#DevOps Project Outline

1) Setting up Terraform to facilitate infrastructure provisioning.

2) Using Terraform to provision Jenkins master, build nodes, and Ansible.

3) Configuring an Ansible server.

4) Employing Ansible to configure Jenkins master and build nodes.

5) Creating a Jenkins pipeline job.

6) Developing a Jenkinsfile from scratch.

7) Configuring SonarQube and Sonar scanner.

8) Executing SonarQube analysis for code quality assessment.

9) Configuring JFrog Artifactory.

10) Creating Dockerfile for containerization.

11) Storing Docker images on Artifactory.

12) Utilizing Terraform to provision a Kubernetes cluster.

13) Creating Kubernetes objects.

14) Deploying Kubernetes objects using Helm.

15) Setting up Prometheus and Grafana using Helm charts.

16) Monitoring the Kubernetes cluster using Prometheus.

==================================================
Part-1 : Prepare Terraform Environment on Windows
===================================================

As part of this, we should setup

Terraform
VS Code
AWSCLI

1) Download terraform the latest version from official website

2) Setup environment variable click on start --> search "edit the environment variables" and click on itUnder the advanced tab, chose "Environment variables" --> under the system variables select path variable and add terraform location in the path variable. system variables --> select path
add new --> terraform_Path

in my system, this path location is C:\terraform_1.3.7

3) Run the below command to validate terraform version

	$ terraform -v
	
4) Install Visual Studio code IDE

5) Download & install AWS cli s/w

6) Create IAM user in AWS & generate access keys

7) Configure IAM user access keys in Environment variables

8) Verify IAM user access keys in cmd

	$ aws configure list


==================================
Part-2 : Launch DevOps Instances
==================================

1) Create Key pair in EC2 (name: dpp.pem)

2) Execute terraform script to Create VPC + EC2 instances

=================================
Part-3 : Setup Ansible Server
=================================

1) Connect to Ansible VM using pem file 

2) Execute below commands to install ansible 

sudo su -
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

3) Create hosts file in /opt 
	
	$ cd /opt
	$ vi hosts

4) Add below content in hosts file 

[jenkins-master]
18.209.18.194

[jenkins-master:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/opt/dpp.pem
 
[jenkins-slave]
54.224.107.148

[jenkins-slave:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/opt/dpp.pem
 

5) Upload dpp.pem file to /opt directory and remove write permissions

	$ chmod 400 dpp.pem
 
6) Test the connectivity 

$ ansible -i hosts all -m ping 

=======================================
Part-4 : Jenkins Master & Slave Setup 
=======================================

1) Connect to jenkins master machine and check jenkins status 

	$ sudo su - 
	$ service jenkins status

2) Execute Ansible Playbook to setup Jenkins in Master Node

	- copy jenkins-setup.yaml into /opt/ 
	
	$ ansible-playbook -i hosts jenkins-master-setup.yaml
	
3) Execute Ansible Playbook to setup Maven in Jenkins Slave 

	- copy jenkins-slave-setup.yaml into /opt 

	$ ansible-playbook -i hosts jenkins-slave-setup.yaml
	
	
### Adding Jenkins Slave Node To Master Node ###

4) Add Credentials 

1. Manage Jenkins --> Manage Credentials --> System --> Global credentials --> Add credentials

2. Provide the below info to add credentials   
   kind: `ssh username with private key`  
   Scope: `Global`     
   ID: `maven_slave`    
   Username: `ubuntu`  
   private key: `dpp.pem key content`  

5) Add slave node to master node

   Follow the below setups to add a new slave node to the jenkins 
   
1. Goto Manage Jenkins --> Manage nodes and clouds --> New node --> Permanent Agent    

2. Provide the below info to add the node   
   Number of executors: `3`   
   Remote root directory: `/home/ubuntu/jenkins`  
   Labels: `maven`  
   Usage: `Use this node as much as possible`  
   Launch method: `Launch agents via SSH`  
        Host: `<Private_IP_of_Slave>`  
        Credentials: `<Jenkins_Slave_Credentials>`     
        Host Key Verification Strategy: `Non verifying Verification Strategy`     
   Availability: `Keep this agent online as much as possible`
   

===========================================
Part-5 : Create First Jenkins Pipeline Job
==========================================

1) Create Jenkins Pipeline 

2) Add Build Stage To Pipeline 

==================================
Part-6 : SonarQube Integration
==================================

1) Create Sonar cloud account on https://sonarcloud.io

2) Generate an Authentication token on SonarQube 

Account --> my account --> Security --> Generate Tokens

Token : 8122aeaf0aabfa0708813588365299c33181de6a

3) On Jenkins create credentials  

Manage Jenkins --> manage credentials --> system --> Global credentials --> add credentials  - Credentials type: Secret text  - ID: sonarqube-key

4) Install SonarQube plugin    

Manage Jenkins --> Available plugins   -->  Search for sonarqube scanner

5) Configure sonarqube server    

Manage Jenkins --> Configure System --> sonarqube server    Add Sonarqube server    - Name: sonar-server    - Server URL: https://sonarcloud.io/    - Server authentication token: sonarqube-key

6) Configure sonarqube scanner    

Manage Jenkins --> Global Tool configuration --> Sonarqube scanner    Add sonarqube scanner    - Sonarqube scanner: sonar-scanner

7) Write sonar-project.properties file  

sonar.verbose=true
sonar.organization=ashokit
sonar.projectKey=ashokit_instalreels
sonar.projectName=instareels
sonar.language=java
sonar.sourceEncoding=UTF-8
sonar.sources=.
sonar.java.binaries=target/classes
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml


8) Create Organization and Project in sonarcloud and configure those details in sonar-project.properties file 

9) push sonar-project.properties file to git repo 

10) Add Sonar stage in jenkins pipeline 

https://docs.sonarsource.com/sonarqube/9.8/analyzing-source-code/scanners/jenkins-extension-sonarqube/

		stage('SonarQube analysis') {
            environment{
                scannerHome = tool 'ashokit-sonarqube-scanner'
            }
			
			steps{
				withSonarQubeEnv('ashokit-sonarqube-server') {
					sh "${scannerHome}/bin/sonar-scanner"
				}
			}
        }

11) Create Quality Gate in sonar cloud and make it default


=========================================
Part-7 : Jfrog Artifactory Integration
=========================================

1) Create Artifactory account
2) Generate an access token with username (username must be your email id)
3) Add username and password under jenkins credentials
4) Install Artifactory plugin
5) Update Jenkinsfile with jar publish stage
