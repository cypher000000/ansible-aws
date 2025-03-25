# Ansible playbook with AWS

Ansible playbook with docker, AWS CLI and AWS

## Project
Create an Ansible playbook that check AWS free tier, create S3 bucket, setup iam and keys, create "build" EC2 ubunutu instance, then build a docker image with maven, create an artifact and push it to the S3 bucket, then create "prod" EC2 ubunutu instance, pull artifact from the s3 bucket and launch docker container with application. And log everything in file.

## How it works

0. Ansible starts playbook, get datetime from OS, automatically login to AWS by AWS CLI credentials, then works by user's tags
1. Check free tier usage for AWS
2. Create (if need) and setup S3 bucket by creating 'artifacts' directory
3. Create (if need) and setup IAM Role and IAM Policy for establish EC2 instances access to S3
4. Upload existing local private key to AWS or create new AWS key pair and save it locally
5. Create (if need) security groups for both EC2 instances then launch (if need) both EC2 instances (build and prod), public ips of created ec2 instances add to ansible dynamic inventory
6. Inside ec2 "build" instance deploys docker, builds the the artifact to S3 bucketmaven-builder docker container, assemble .war artifact, tags the artifact with datetime and push the artifact to S3 bucket
7. Inside ec2 "prod" instance deploys docker, pulls the artifact from S3 bucket and run the Tomcat-container with new artifact
8. Test if app reachable and then display application URL
9. You can check the app by ip_prod:80/demo/Hello but only from ip where Ansible playbook was initiated. (You can add different cidr_ip inside ec2_security_group in playbooks/5_setup_ec2_instances.yml)
10. Everything logged in file if you used "log" tag

## Versions

- Ansible 2.17.4
- Git 2.34.1
- AWS CLI 2.18.0
- all installed on Ubuntu 22.04
  
## How to make this work

On your VM/PC/EC2/etc:

- Register on AWS and create an account (if you doesnt have any). Make sure that it has all permissions to create ec2 instance/security groups/IAM/upload ssh key and work with S3
- Setup Ansible and AWS CLI, login in your AWS account

- (Optional) If you want, you can create ssh key for ansible user (aws-key-1.pem + aws-key-1.pem.pub) and then upload it in AWS. also How to generate ssh-key code below.
 How to generate ssh-key:
``` bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/test-aws-ansible-key-1.pem
chmod 400 ~/.ssh/test-aws-ansible-key-1.pem
```

## Setup Ansible playbook
Everything triggered by tags, so use them

1. Clone repo:
``` bash
git clone https://github.com/cypher000000/ansible-aws.git 
```
- Specify all your variables in 'host_vars/all.yml'
2. Start playbook, using tags, TLDR:
``` bash
ansible-playbook -i "localhost," --connection=local site.yml --tags "log,s3_setup,iam_setup,key_management,create_key,ec2_instances,build_server,prod_server"
```
or you can select your tags:
``` bash
ansible-playbook -i "localhost," --connection=local site.yml --tags "tag,tag,tag"
```
- All tags:
  - "log"
    - 1
  - "msg"
    - 2
  - "free_tier_check"
    - 3
  - "s3_setup"
    - 4
  - "iam_setup"
    - 5
  - "key_management,upload_key" or "key_management,create_key"
    - 6
  - "ec2_instances,build_server,prod_server"
    - 7
## Credits for demo artifact app
@tongueroo for https://github.com/tongueroo/demo-java
