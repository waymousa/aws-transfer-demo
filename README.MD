# AWS tramnsfer Demo

This CloudFormation script builds out an AWS Transfer environment with KMS bucket encryption.  Trust me when I say you do not want to do this manually.

# 1. Create the Transfer Server
The purpose of this CloudFormation template is to enable a Transfer Server, CloudWatch logging, VPC, EIP, CC Endfpoint and Subnets needed to set up the environment for SFTP transfers.
Open CLoudForemat and run the following script *1-aws-transfer-server-stack.yaml*

# 2. Enable The Internet Endpoint
The purpose of this step is to enable access to the SFTP Server from the Internet.  You will set up the Security Group against the VPC Endpoint to control who can access the site.  Optionally, you can import an existing ssh server public key to help identify your server to the clients.

# 3. Set up Users and Bucket resources
The purpose of this step is to enable the users of the site.  You will need to create some ssh keys for the server and users.  This step also creates the bucket and IAM roles necessary for your users and the AWS Transfer Server to process requests.
Run the following CloudFormation script in your target account *2-aws-transfer-user-stack.yaml*