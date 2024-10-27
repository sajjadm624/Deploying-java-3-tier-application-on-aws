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

   ```bash
   sudo yum install amazon-ssm-agent
   sudo systemctl start amazon-ssm-agent
   sudo systemctl enable amazon-ssm-agent
   ```
