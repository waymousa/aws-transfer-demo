# AWS Transfer Demo

This [CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html "CloudFormation") script builds out an demo SFTP Enabled [AWS Transfer Server](https://docs.aws.amazon.com/transfer/latest/userguide/create-server-sftp.html "AWS Transfer Server") environment with [KMS bucket encryption with CMK](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html "KMS bucket encryption with CMK").  Trust me when I say you do not want to do this manually.  

### POINT TO NOTE
The code is for demo purposes only and should not be considred production ready.  Feel free to fork the repo and use it as a starting point for your own production stable code.

# A. Pre-requisite activities
This project contains a CloudFormation template that sets up the SFTP Transfer server and 2 sample users.  Now the topic of how one manages ssh keys for these users is a matter of policy ijn youir organisation, however I would strongly recommend that you take the same approach as gitbu and other public repos, whjich is to insist that your customers manage their ssh key pairs tmelseves and opnly send you the public key.  This pushes the responsibility for private key management back on the customer.  To this end, the CloudFormation template has a parameter for each user to supply a ssh public key.  This is not a security risk because its only the public key that is needed.  If you are building this out as a demo for your own use then you can generate the key pairs yourself using ssh-keygen and then put the public keys in the template as default values or at run time which is what I did.  An example of how you can do this is shown below
```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/swaym/.ssh/id_rsa): data-transfer-user3
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in data-transfer-user3.
Your public key has been saved in data-transfer-user3.pub.
The key fingerprint is:
SHA256:2WnO/EARidHGsMOF33plso0DrM0oqbWUOm/9nxoJc4U swaym@LHR14-CG73858CM
The key''s randomart image is:
+---[RSA 2048]----+
|        oBo.     |
|       .oo=o     |
|        ++E..    |
|         +++o o  |
|       oS=*o B   |
|      * oX+.= .  |
|     = +  B. .   |
|    + o .  + .   |
|     +.  .oo+    |
+----[SHA256]-----+
$ cat data-transfer-user3.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDu7+gybjvDOAf2OzbJb5qLomyQGz27n7TVR4BUJe6kqzcakxwhBiAwxab65BN5jLFuZRPV5qs2P48nUJp4PmLXwVmBIme1UNXUVte3dJRGttnbCmCBdS0HhyX3swDWosaRxUHrQCQ/0GzIvjZmdFF6eFSKJ63cZ1GAcIsVUZKW9F1b446QDCFmsedGW/hqOM8Kgn9h8WQqJhaHGIeg0HmU9/cdSOB9cdoIXEgBcAdVzRUyAQloKR3+CJI2P7dTk9K5NqYhr1opvsbF81eecsLQaVkl48lkWy8lorOG8JkUgz56MvircV2s/9kDa9Np8ZM2/gIGHfG2QO3phEcVVbtB
```

# B. Create the Transfer Server
The purpose of this CloudFormation template is to enable a Transfer Server, CloudWatch logging, VPC, EIP, Internet facing Endpoint and Subnets needed to set up the environment for SFTP transfers.  It also creates an S3 Bucket, CMK, IAM policies and some sample users.  

To run the script, open AWS Console in the account you wish to test this in and uswe the following CloudFormation Template *1-aws-transfer-server-stack.yaml*.  

When you run the script, you will see parameters for the sample user name and ssh public key.  The username parameter is used for both the name of the user and the name of the home directory that you will need to create in the S3 bucket.  

It is possible to generate the folders in the bucket automatically as part of the CloudFormation template but requires the addition of custom CloudFoprmation targets as described in this StackOverflow thread https://stackoverflow.com/questions/36917947/create-folder-inside-s3-bucket-using-cloudformation.  

Full automation of user creation and ongoing management is beyond the scope of this demo, but I would probaboly not recommend you do that task as a CloudFormation activity.  Instead, it makes sense to ingertare your ISMS with the solution usign the Transfer CLI/API.

# 2. Enable Scope Down Policy
The scope-down policy for users needs to be added to the user object as a JSON blob.  While you can create the scope-down policy as an IAM object, its is not possible to reference it by arn in the CloudFormation template so you need to update the user records manually after they have ben created.  You can do this in the console, or you can attach the policy to the user objects using the following CLI command:
```bash
aws transfer update-user --server-id server --user-name user --policy \
   "$(aws iam get-policy-version --policy-arn policy --version-id version --output json)"
```