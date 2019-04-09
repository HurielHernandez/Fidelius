# Quick Start

Fidelius includes a simple way to get Fidelius running locally to get you started.

Before you start you need to make sure the right permissions are set in order for the script to create the resources
required to get Fidelius running locally.  The script is able to 

- Create a DynamoDb table with the proper format to store secrets
- Build the required docker containers
- Start Fidelius locally with some services mocked talking to your DynamoDb table in order for you to be allowed to 
experience all of Fidelius features.

Demo Features

- Login as a Dev user
- Login as an Ops user  
- Login as a Master user
- Create secrets
- View secrets
- Update secrets
- Delete secrets

## Prerequisites

Fidelius requires the following tools to be installed and on your $PATH variable:

- [Java 8+](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- [Maven 3+](https://maven.apache.org/install.html)
- [NPM 3+](https://www.npmjs.com/get-npm)
- [Docker](https://docs.docker.com/install/)
- [AWS-CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

### Step 1 - Create a KMS key
 Create a KMS key with an alias `credstash` in AWS KMS.  Reference can be found [here](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html)
 
### Step 2 - Create User Role

Create a user role that will be used to run the setup script.  This role will also assume the application role.  This role should have the following permissions:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "dynamodb:CreateTable",
                "dynamodb:DescribeTable"
            ],
            "Effect": "Allow",
            "Resource": "arn:aws:dynamodb:<REGION>:<ACCOUNT_NUMBER>:table/credential-store"
        },
        {
            "Action": [
                "dynamodb:ListTables"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```
### Step 3 - Create Cross Account Role
 
Create a AWS role that your user role can assume.  This role should be called `Cross_Account_Fidelius`.  This can be changed
in the future.  The role needs to have this IAM permissions.    
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:Encrypt",
                "kms:GenerateDataKey",
                "dynamodb:PutItem",
                "dynamodb:BatchWriteItem",
                "dynamodb:GetItem",
                "dynamodb:Scan",
                "dynamodb:Query",
                "dynamodb:UpdateItem"
            ],
            "Resource": [
                "arn:aws:kms:<REGION>:<ACCOUNT_NUMBER>:key/<KMS_KEY_ID>",
                "arn:aws:dynamodb:<REGION>:<ACCOUNT_NUMBER>:table/credential-store"
            ]
        }
    ]
}
```

### Step 4 - Grant Assume Role 
Grant permission for your user role from step 2 to assume into `Cross_Account_Fidelius` role create on Step 3.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::<ACCOUNT_NUMBER>:role/Cross_Account_Fidelius"
        }
    ]
}

```

### Step 5 - Update Trust Relationship
Add your user role to Trust Relationship on `Cross_Account_fidelius` 
```json
 arn:aws:iam::AWSACCOUNTID:role/YOUR_USER_ROLE
```

### Step 6 - Set Environment Variables
Set environment variables

If you need a proxy, set environment variable `http_proxy`
```bash 
    export http_proxy=proxy.com:80
```

Set the location of your `AWS_DIRECTORY` that contains your credentials
```bash 
    export AWS_DIRECTORY=~/.aws
```

Set your `AWS_ACCOUNT_NUMBER` that will be used to create the resources needed for Fidelius to launch
```bash
    export AWS_ACCOUNT_NUMBER=12345678910
```

### Step 7 - Run Start Script
Run the following script that will build all the Fidelius code/containers and bring
Fidelius up locally.

Refresh your AWS tokens if you need to, otherwise run the start script.
```bash
bash start.sh
```


You should see the following screens in this order: 

```bash
-------------------------------------------------------
|         Building base Fidelius containers           |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|         Building Fidelius demo services             |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|         Building Fidelius setup container           |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|         Building Fidelius backend service           |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|         Building Fidelius UI                        |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|         Building Fidelius containers                |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|          Starting Fidelius Setup                     |
-------------------------------------------------------
.
.
.
-------------------------------------------------------
|          Starting Fidelius Local Environment         |
-------------------------------------------------------
.
.
.

-------------------------------------------------------
|              Fidelius User Endpoints                |
|                                                     |
|         DEV user -      https://localhost:443       |
|         OPS user -      https://localhost:444       |
|         MASTER user -   https://localhost:445       |
|                                                     |
-------------------------------------------------------

```

### Step 8 - Navigate to Fidelius

Navigate to the links above to experience Fidelius Secrets Manager.  


Fidelius User Endpoints

| ROLE   	| URL                   	|
|--------	|-----------------------	|
| Dev    	| https://localhost:443 	|
| Ops    	| https://localhost:444 	|
| Master 	| https://localhost:445 	|


### Step 9 - Stop Fidelius

To stop Fidelius service you can run the stop script

```
bash stop.sh
```

## Next Steps
After having Fidelius running locally with default configurations, you might want to configure the steps so that you
can run or deploy Fidelius for your organization.

The application configurations can be found at [Configuration](prequisites/configuration.md)