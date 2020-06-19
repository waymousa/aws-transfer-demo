# AWS Transfer Demo

This [CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html "CloudFormation") script builds out an demo SFTP Enabled [AWS Transfer Server](https://docs.aws.amazon.com/transfer/latest/userguide/create-server-sftp.html "AWS Transfer Server") environment with [KMS bucket encryption with CMK](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html "KMS bucket encryption with CMK").  Trust me when I say you do not want to do this manually.  

### POINT TO NOTE
The code is for demo purposes only and should not be considered production ready.  Feel free to fork the repo and use it as a starting point for your own production stable code.

# A. Pre-requisite activities
This project contains a CloudFormation template that sets up the SFTP Transfer server and 2 sample users.  The topic of how one manages ssh keys for these users is a matter of policy in your organisation, however I would strongly recommend that you take the same approach as github and other public repos which is to require your customers manage their own ssh key pairs and only send you the public key.  This pushes the responsibility for private key management back on the customer.  To this end, the CloudFormation template has a parameter for each user to supply a ssh public key.  This is not a security risk because its only the public key that is needed.  If you are building this out as a demo for your own use then you can generate the key pairs yourself using ssh-keygen and then put the public keys in the template as default values or at run time which is what I did.  An example of how you can do this is shown below.
```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/swaym/.ssh/id_rsa): data-transfer-user1
Enter passphrase (empty for no passphrase): <your-password-here>
Enter same passphrase again: <your-password-here>
Your identification has been saved in data-transfer-user1.
Your public key has been saved in data-transfer-user1.pub.
The key fingerprint is:
SHA256:2WnO/EARidHGsMOF33plso0DrM0oqbWUOm/9nxoJc4U
The key\'s randomart image is:
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
You can now put the public key into the CloudFormation Template as a default parameter or you can insert it during the deployment time.  Do not expose your private key and keey is password protected.

# B. Create the Transfer Server
The purpose of this CloudFormation template is to enable a Transfer Server, CloudWatch logging, VPC, EIP, Internet facing Endpoint and Subnets needed to set up the environment for SFTP transfers.  It also creates an S3 Bucket, CMK, IAM policies and some sample users.  To run the script, open AWS Console in the account you wish to build the environment within and use the following CloudFormation Template **1-aws-transfer-server-stack.yaml**.  When you run the script, you will see parameters for the sample user name and ssh public key.  The username parameter is used for both the name of the user and the name of the home directory.  The Cloudformation script does not create the S3 folder for you so you will need to do that manually after the script has run.  

1. Navigate to the **Cloudformation** service in the **AWS Console**.
2. Click on the **Create Stack** button and select **with new resources (standard)** from the menu.
3. In the **Specify template** window, select **upload a template file**.
4. Click on the **Choose file** button and then select the template file **1-aws-transfer-server-stack.yaml**.  Click on the **open** button.
5. Click on the **next** button to continue.
6. Type in **data-transfer-stack** for the Stack Name.  Review the other parameters for correctness.  Make sure that you have the right details for the **user** names, **ssh public keys**, **VPC and Subnet** CIDR ranges, as well as the CIDR range for the **security group**.  Click **next** button to continue.
7. At the **stack options** page click on **next** to continue.
8. In the **review stack** page, click on the checkbox for **I acknowledge that AWS CloudFormation might create IAM resources** and then click on the **Create stack** button.  The stack will now run for about 5-10 minutes.  **Pause here** until the stack deployment is completed.
9. Navigate to the **S3 Service** page in the **AWS Console**.
10.  Locate the bucket created during the stack deployment.  It should have a name beginning with **data-transfer-stack** followed by a unique ID.
11.  Create two folders in the root of the bucket, using the names of each user that you supplied in the stack paramaters.  By default these are **user1** and **user2**.  Click on the **create folder** button.  Type in the name of the user.  Select **AWS-KMS** as the encryption type and select the **data-transfer-cmk** as the key.  Click on the **save** button for each bucket as you create it.  Repeat this proces for each user.

### POINT TO NOTE
It is possible to generate the folders in the bucket automatically as part of the CloudFormation template but requires the addition of custom CloudFormation targets as described in this StackOverflow thread [create folder inside S3 bucket using Cloudformation](https://stackoverflow.com/questions/36917947/create-folder-inside-s3-bucket-using-cloudformation "create folder inside S3 bucket using Cloudformation").  Full automation of user creation and ongoing management is beyond the scope of this demo, but I would probaboly not recommend you do that task as a CloudFormation activity.  Instead, it makes sense to integrate your ISMS with the solution using the Transfer CLI/API.

# C. Update the Security Group
The purpose of this step is to change the Transfer Server Endpoint to use a customer Security Group that was created by the CloudFormation template.  This allows you to secure your Transfer Server site as described in this blog post [Use IP whitelisting to secure your AWS Transfer for SFTP servers](https://aws.amazon.com/blogs/storage/use-ip-whitelisting-to-secure-your-aws-transfer-for-sftp-servers/ "Use IP whitelisting to secure your AWS Transfer for SFTP servers").  The Security Group created by the CloudFormation Script is open to the world by default.

1. Navigate to the AWS Transfer Services page in the AWS Console.
2. Open the transfer server you have just created.
3. Select the **VPC Endpoint** by clicking on the url link.
4. Select the **Security Group** tab and then click on the **edit security groups** button.
5. Ensure that only the security group labled **data-transfer-stack-SecurityGroup1** is selected.  Click on the **save** button. 

# D. Enable Scope Down Policy
The scope-down policy for users needs to be added to the user object as a JSON blob.  While you can create the scope-down policy as an IAM object, its is not possible to reference it by arn in the CloudFormation template so you need to update the user records manually after they have ben created.  You can do this in the AWS Console, or you can attach the policy to the user objects using the following CLI command as a guide.
```bash
aws transfer update-user --server-id server --user-name user --policy \
   "$(aws iam get-policy-version --policy-arn policy --version-id version --output json)"
```

1. Navigate to the AWS Transfer Services page in the AWS Console.
2. Open the transfer server you have just created.
3. Click on the user name, then in the **user copnfiguration** window click on the **edit** button.
4. In the **policy** feild, select the radio button **select policy from IAM**.  Select the policy starting with the name **data-transfer-stack-scope-down-policy** and then click on the **save** button.

# E. Test
Follow the testing procedure outlined at this link for specific instructions related to your [file transfer client](https://docs.aws.amazon.com/transfer/latest/userguide/getting-started-use-the-service.html, "file transfer client.")

# F. Delete the stack
You can delete the stack once you have finished playing with it.  First, shutdown the Transfer Server to prevent usaers from uploading new files.  Next, empty the S3 bucket.  Last, delete the stack to delete the resources.  The CMK will be disabled then deleted after 7 days.