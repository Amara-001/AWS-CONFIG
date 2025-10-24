# 🛡️ AWS Config Compliance Automation — NIST CSF Framework  

### 👩🏾‍💻 Author: Golde Amie Wamuo  
*Cybersecurity Analyst | Cloud Security Enthusiast | CyberGirls Alumna*  

---

## 📋 Project Summary  

| 🔹 **Objective** | Automate security compliance checks and remediation using AWS Config, NIST CSF, and Bash scripting |
|------------------|--------------------------------------------------------------------------------------------|
| 🧠 **Focus Areas** | Cloud Security • Compliance Automation • AWS Config • Infrastructure as Code (IaC) |
| 🧩 **Key Tools** | AWS Config • CloudFormation • CloudTrail • VPC Flow Logs • AWS CLI • Bash |
| ⚙️ **Duration** | 2 hours (end-to-end deployment, evaluation, and remediation) |
| 📊 **Initial Compliance** | 66% |
| 🟢 **Final Compliance** | **90%** |
| 🏗️ **Framework Applied** | NIST Cybersecurity Framework (CSF) |
| 💡 **Outcome** | Automated detection and remediation of noncompliant cloud resources within a sandbox environment. |

---

## 🧭 Overview  

This mini-project demonstrates how I designed and deployed a **compliance monitoring and remediation environment** on AWS using:  

- **AWS Config**  
- **NIST Cybersecurity Framework (CSF)** Conformance Pack  
- **CloudFormation** for Infrastructure-as-Code (IaC)  
- **Bash Automation** for quick remediation  

The goal was to simulate a real-world enterprise cloud environment, evaluate it against compliance benchmarks, and automatically remediate security gaps to raise compliance from **66% to 90%**.

---

## 🧱 Step 1: Create Lab Infrastructure Using CloudFormation  

### **Resources created**
- 1 S3 Bucket (`goldeamiebucket103`)  
- 2 EC2 instances (`webserver` and `mgtserver`)  
- 1 Security Group (SSH restricted to my IP)  
- 1 custom VPC (`default`) and subnet  

### **CloudFormation Template Used**
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  Golde's AWS Config Lab - Creates an S3 bucket (goldeamiebucket103) and two EC2 instances
  (webserver and mgtserver) in the existing default VPC using Amazon Linux 2023 AMI.

Parameters:
  MyIP:
    Type: String
    Description: Enter your public IP address (e.g., 192.168.20.164/32) for SSH access
    Default: 0.0.0.0/0

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0341d95f75f311023

Resources:
  GoldeS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: goldeamiebucket103
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      Tags:
        - Key: Name
          Value: goldeamiebucket103

  DefaultVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: default

  DefaultSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref DefaultVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: us-east-1a
      Tags:
        - Key: Name
          Value: default-subnet

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH access from your IP only
      VpcId: !Ref DefaultVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref MyIP
      Tags:
        - Key: Name
          Value: golde-security-group

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      KeyName: GoldeKeyPair
      ImageId: !FindInMap [RegionMap, us-east-1, AMI]
      NetworkInterfaces:
        - AssociatePublicIpAddress: true
          DeviceIndex: 0
          SubnetId: !Ref DefaultSubnet1
          GroupSet:
            - !Ref InstanceSecurityGroup
      Tags:
        - Key: Name
          Value: webserver
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl enable httpd
          systemctl start httpd
          echo "<h1>Welcome to Golde's WebServer</h1>" > /var/www/html/index.html

  MgtServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      KeyName: GoldeKeyPair
      ImageId: !FindInMap [RegionMap, us-east-1, AMI]
      NetworkInterfaces:
        - AssociatePublicIpAddress: true
          DeviceIndex: 0
          SubnetId: !Ref DefaultSubnet1
          GroupSet:
            - !Ref InstanceSecurityGroup
      Tags:
        - Key: Name
          Value: mgtserver
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y aws-cli jq
          echo "Management Server Ready" > /home/ec2-user/mgtserver.txt

Outputs:
  S3BucketName:
    Description: The created S3 bucket
    Value: !Ref GoldeS3Bucket

  WebServerPublicIP:
    Description: Public IP of WebServer
    Value: !GetAtt WebServer.PublicIp

  MgtServerPublicIP:
    Description: Public IP of MgtServer
    Value: !GetAtt MgtServer.PublicIp





