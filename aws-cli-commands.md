### These commands are used in the bootcamp AWS CLI lectures


#### AWS Configure
 
```sh
aws configure
aws configure set aws_access_key_id  "YOUR_ACCESS_KEY_ID"
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY= "YOUR_SECRET_ACCESS_KEY"
```
  
#### AWS EC2 Service

```sh
aws ec2 help
aws ec2 describe-security-groups --group-ids "YOUR_GROUP_ID"
aws ec2 describe-vpcs
aws ec2 create-security-group --group-name "YOUR_GROUP_NAME" --description "ENTER_DESCRIPTION" --vpc-id "YOUR_VPC_ID"
aws ec2 describe-subnets
aws ec2 describe-instances
aws ec2 describe-instances --filters Name="ENTER_FILTER_NAME",Values="ENTER_VALUE" --query "Reservations[].Instances[].InstanceId"
aws ec2 run-instances --image-id "YOUR_AMI_ID" --count "ENTER_NUMBER" --instance-type "YOUR_INSTANCE_TYPE" --key-name "YOUR_KEY_NAME" --security-group-ids "YOUR_SECURITY_GROUP_ID" --subnet-id "YOUR_SUBNET_ID"
aws ec2 authorize-security-group-ingress --group-id "YOUR_GROUP_ID" --protocol "ENTER_PROTOCOL" --port "YOUR_PORT_NUMBER" --cidr "YOUR_CIDR_BLOCK"
aws ec2 create-key-pair --key-name "YOUR_KEY_NAME" --query "ENTER_QUERY" --output text > "YOUR_PEM_FILE"
```

#### AWS IAM Service
  
```sh
aws iam help
aws iam create-group --group-name "YOUR_GROUP_NAME"
aws iam create-user --user-name "YOUR_USER_NAME"
aws iam add-user-to-group --user-name "YOUR_USER_NAME" --group-name "YOUR_GROUP_NAME"
aws iam get-group --group-name "YOUR_GROUP_NAME"
aws iam attach-user-policy --user-name "YOUR_USER_NAME" --policy-arn "YOUR_POLICY_ARN"
aws iam list-policies --query `Policies[?PolicyName=="`YOUR_POLICY_NAME`"].Arn` --output text
aws iam attach-group-policy --group-name "YOUR_GROUP_NAME" --policy-arn "YOUR_POLICY_ARN"
aws iam list-attached-group-policies --group-name "YOUR_GROUP_NAME"
aws iam create-login-profile --user-name "YOUR_USER_NAME" --password "YOUR_PASSWORD" --password-reset-required
aws iam get-user --user-name "YOUR_USER_NAME"
aws iam create-policy --policy-name "YOUR_POLICY_NAME" --policy-document "YOUR_FILE_NAME"
aws iam create-access-key --user-name "YOUR_USER_NAME"
```
