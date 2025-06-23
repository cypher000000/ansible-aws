# Ansible playbook with AWS

Ansible playbook with Docker, AWS CLI and AWS

## Project
Create an Ansible playbook that check AWS free tier, create S3 bucket, setup iam and keys, create "build" EC2 ubuntu instance, then build a docker image with maven, create an artifact and push it to the S3 bucket, then create "prod" EC2 ubuntu instance, pull artifact from the s3 bucket and launch docker container with application. And log everything in a file.

## How it works

0. Ansible starts playbook, gets datetime from OS, automatically logs in to AWS by AWS CLI credentials, then works by user's tags
1. Check free tier usage for AWS
2. Create (if needed) and setup S3 bucket by creating 'artifacts' directory
3. Create (if needed) and setup IAM Role and IAM Policy for establish EC2 instances' access to S3
4. Upload existing local private key to AWS or create new AWS key pair and save it locally
5. Create (if needed) security groups for both EC2 instances then launch (if need) both EC2 instances (build and prod), public IPs of created EC2 instances add to Ansible dynamic inventory
6. Inside EC2 "build" instance deploys docker, builds the artifact to S3 bucket maven-builder docker container, assemble .war artifact, tags the artifact with datetime, and push the artifact to S3 bucket
7. Inside EC2 "prod" instance deploys docker, pulls the artifact from S3 bucket, and run the Tomcat-container with new artifact
8. Test if app is reachable and then display application URL
9. You can check the app by ip_prod:80/demo/Hello but only from IP where Ansible playbook was initiated. (You can add different cidr_ip inside ec2_security_group in playbooks/5_setup_ec2_instances.yml)
10. Everything logged in file if you used "log" tag
11. Don't forget to clean everything in AWS manually by yourself, I'll write cleaning playbooks later

## Versions

- Ansible 2.17.4
- Git 2.34.1
- AWS CLI 2.18.0
- all installed on Ubuntu 22.04
  
## How to make this work

On your VM/PC/EC2/etc:

- Register on AWS and create an account (if you doesnt have any). Make sure that it has all permissions to create EC2 instance/security groups/IAM/upload ssh key and work with S3
- Set up Ansible and AWS CLI, login in to your AWS account

- (Optional) If you want, you can create ssh key for Ansible user (aws-key-1.pem + aws-key-1.pem.pub) and then upload it in AWS. Also how to generate ssh-key code below.
 How to generate ssh-key:
``` bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/test-aws-ansible-key-1.pem
chmod 400 ~/.ssh/test-aws-ansible-key-1.pem
```

## Setup Ansible playbook
Everything is triggered by tags, so use them

1. Clone repo:
``` bash
git clone https://github.com/cypher000000/ansible-aws.git
cd ansible-aws
```
- Specify all your variables in 'host_vars/all.yml'
2. Start playbook, using tags, TLDR:
``` bash
ansible-playbook -i "localhost," --connection=local site.yml --tags "log,msg,s3_setup,iam_setup,key_management,create_key,ec2_instances,build_server,prod_server"
```
or you can select your tags:
``` bash
ansible-playbook -i "localhost," --connection=local site.yml --tags "tag,tag,tag"
```
- All tags definition:
  - "log"
    - Loging every step in log_file_path with timestamp.
  - "msg"
    - Messaging every step in Ansible output.
  - "free_tier_check"
    - Checking AWS Free Tier usage for EC2, EBS, S3, Data Transfer, Public IPS; showing if you have any high usage.
  - "s3_setup"
    - Creating (if needed) S3 bucket "s3_bucket" and dir "artifacts" inside.
  - "iam_setup"
    - Creating (if need) AWS IAM policy "iam_policy_name" and role "iam_role_name" with access from EC2 to S3 bucket "s3_bucket".
  - "key_management,upload_key" or "key_management,create_key" - you must specify one of this variations
    - "create_key" tag: Creating EC2 Key Pair "aws_key_name" and save it locally in "aws_ssh_private_key_file".
    - "upload_key" tag: Creating EC2 Key "aws_key_name" by existing public key "aws_ssh_private_key_file.pub".
  - "ec2_instances,build_server,prod_server" - you must specify all 3 exactly like this
    - "ec2_instances" tag: Creating (if need) and Configure 2 Dynamic Security Group (only 1 IP address and ports 22/80 for public) and Create and Configure 2 EC2 Instances (build-server and prod-server) with type "instance_type" and "image_id", role "iam_role_name" and VPC "vpc_id", then add both instances to in-memory dynamic Ansible inventory.
    - "build_server" tag: Configuring Build Server by "build" Ansible role - install docker, build and run docker image to build maven artifact, upload WAR artifact to S3 "s3_bucket" with datetime tag.
    - "prod_server" tag: Configuring Prod Server by "deploy" Ansible role - install docker, download WAR artifact from S3 "s3_bucket", build and run docker image with the built artifact from S3, then output application access URL.
## Credits for demo artifact app
@tongueroo for https://github.com/tongueroo/demo-java
