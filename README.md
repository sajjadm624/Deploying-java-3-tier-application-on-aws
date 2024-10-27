# Deploying-java-3-tier-application-on-aws
This project will teach us how to deploy a scalable, highly available, and secure Java application using a 3-tier architecture. 

## Table of Contents

1. [Goal](#goal)
2. [Pre-Requisites](#pre-requisites)
3. [Pre-Deployment](#pre-deployment)
4. [VPC Deployment](#vpc-deployment)
5. [Maven (Build)](#maven-build)
6. [3-Tier Architecture](#3-tier-architecture)
7. [Application Deployment](#application-deployment)
8. [Post-Deployment](#post-deployment)
9. [Validation](#validation)

# Projects Overview
## Goal

The primary objective of this project is to deploy a scalable, highly available, and secure Java application using a 3-tier architecture. The application will be hosted on AWS, utilizing various services like EC2, RDS, and VPC to ensure its availability, scalability, and security. It will be accessible to end-users via the public Internet.

## Pre-Requisites

Before starting the deployment, ensure you have the following:

1. **AWS Free Tier Account**:
   - Sign up for an [Amazon Web Services (AWS) Free Tier account](https://aws.amazon.com/free/).
   - Set up the AWS CLI on your local machine.
   - Configure your CLI with `aws configure`.

2. **GitHub Account and Repository**:
   - Create a [GitHub account](https://github.com/join) if you don't have one.
   - Fork the Java source code from the [Java-Login-App repository](https://github.com/NotHarshhaa/DevOps-Projects/blob/master/DevOps-Project-01/Java-Login-App) to your GitHub account.

3. **SonarCloud Account**:
   - Create an account on [SonarCloud](https://sonarcloud.io/) for static code analysis and code quality checks.
   - Generate a SonarCloud token for integration with your GitHub repository.

4. **JFrog Cloud Account**:
   - Sign up for a [JFrog Cloud account](https://jfrog.com/start-free/).
   - Set up a Maven repository in JFrog to store your build artifacts.


## Pre-Deployment

### Create Global AMI (Amazon Machine Image)

Creating a Global AMI involves installing necessary agents and software on an EC2 instance, which will be used to create custom AMIs for different components.

1. **Install AWS CLI**:
   - Install AWS CLI by following the instructions [here](https://aws.amazon.com/cli/).

   ```bash
   curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
   sudo installer -pkg AWSCLIV2.pkg -target /
   aws --version
   ```
2. **Install CloudWatch Agent**:
   - Install CloudWatch Agent to monitor your EC2 instances.

   ```bash
   sudo yum install amazon-cloudwatch-agent
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start
   ```

3. **Install AWS Systems Manager (SSM) Agent**:
   - The SSM Agent is necessary for managing EC2 instances via AWS Systems Manager.

### Create Golden AMIs

Golden AMIs will be created for the different tiers (Nginx, Tomcat, Maven) of the architecture.

1. **For Nginx**:
   - Launch an EC2 instance and install Nginx:

   ```bash
   sudo amazon-linux-extras install nginx1.12
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

   - Create a custom CloudWatch metric for memory usage:

   ```bash
   #!/bin/bash
   while true; do
     memory_usage=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
     aws cloudwatch put-metric-data --metric-name MemoryUsage --namespace Custom --value $memory_usage --dimensions InstanceId=$(curl http://169.254.169.254/latest/meta-data/instance-id)
     sleep 60
   done &
   ```

2. **For Apache Tomcat**:
   - Launch another EC2 instance and install Apache Tomcat:

   ```bash
   sudo yum install java-22-amazon-corretto-devel.x86_64
   # For jvm library
   wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.0/bin/apache-tomcat-11.0.0.tar.gz
   sudo tar -xvzf apache-tomcat-11.0.0.tar.gz -C /opt/
   sudo ln -s /opt/apache-tomcat-11.0.0 /opt/tomcat
   sudo sh /opt/tomcat/bin/startup.sh
   ```

   - Configure Tomcat as a systemd service:

   ```bash
   sudo nano /etc/systemd/system/tomcat.service
   ```

   Add the following content:

   ```ini
   [Unit]
   Description=Apache Tomcat Web Application Container
   After=network.target

   [Service]
   Type=forking

   Environment=JAVA_HOME=/usr/lib/jvm/jre
   Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
   Environment=CATALINA_HOME=/opt/tomcat
   Environment=CATALINA_BASE=/opt/tomcat
   Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
   Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

   ExecStart=/opt/tomcat/bin/startup.sh
   ExecStop=/opt/tomcat/bin/shutdown.sh

   User=tomcat
   Group=tomcat
   UMask=0007
   RestartSec=10
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

   - Enable and start the Tomcat service:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start tomcat
   sudo systemctl enable tomcat
   ```

3. **For Apache Maven Build Tool**:
   - Install Maven, Git, and JDK 11 on a separate EC2 instance:

   ```bash
   sudo yum install git
   sudo yum install java-22-amazon-corretto-devel.x86_64
   wget https://mirrors.ocf.berkeley.edu/apache/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz
   sudo tar -xvzf apache-maven-3.8.4-bin.tar.gz -C /opt/
   sudo ln -s /opt/apache-maven-3.8.4 /opt/maven
   ```

   - Update the system PATH:

   ```bash
   echo "export M2_HOME=/opt/maven" | sudo tee -a /etc/profile.d/maven.sh
   echo "export PATH=\$M2_HOME/bin:\$PATH" | sudo tee -a /etc/profile.d/maven.sh
   source /etc/profile.d/maven.sh
   ```

   - Verify the installation:

   ```bash
   mvn -version
   ```
