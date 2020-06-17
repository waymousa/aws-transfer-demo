# AWS tramnsfer Demo

This CloudFormation script builds out an AWS Transfer environment with KMS bucket encryption.  Trust me when I say you do not want to do this manually.

# 1. Create the Transfer Server
The purpose of this CloudFormation template is to enable a Transfer Server, CloudWatch logging, VPC, EIP, CC Endfpoint and Subnets needed to set up the environment for SFTP transfers.
Open CLoudForemat and run the following script *1-aws-transfer-server-stack.yaml*

# 2. Enable Scope Down Policy
It is not possible to sert up the scope down policy in the CloudFormation script when you need to dynamically set up the CMK for the bucket.  There is a policy that is created that you need to attache to the user objects uusing the following CLI command:
aws transfer update-user --server-id server --user-name user --policy \
   "$(aws iam get-policy-version --policy-arn policy --version-id version --output json)"