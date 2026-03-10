# Serverless Slack ChatOps Bot: AWS EC2 Provisioner



## 🚀 Overview
A serverless ChatOps integration that allows engineering teams to provision AWS EC2 instances directly from Slack. By typing `/ec2`, users are presented with an interactive Slack Modal to configure their instance (OS, Type, Storage). The bot handles the infrastructure provisioning in the background and securely delivers the Public IP, SSH instructions, and the newly generated `.pem` private key directly to the user's Direct Messages.

## ✨ Features
* **Serverless Architecture:** Fully hosted on AWS Lambda via Function URLs (Zero idle server costs).
* **Interactive Modals:** Uses Slack's Block Kit UI to gather instance configurations via dropdowns and text inputs.
* **Timeout-Proof (Lazy Listeners):** Utilizes the Slack Bolt framework's lazy listener architecture to acknowledge Slack's strict 3-second timeout requirement while performing heavy AWS Boto3 tasks in the background.
* **Automated Key Management:** Automatically generates a unique RSA Key Pair for the instance and securely uploads the `.pem` file to the user via a private Slack DM.
* **Smart Polling (Boto3 Waiters):** Pauses background execution until the EC2 instance is fully running to retrieve and return the dynamically assigned Public IP address.
* **Standardized Security Group:** Automatically attaches a pre-configured Security Group to all newly launched instances to ensure standard firewall rules (like SSH access on port 22) are applied by default.

## 🛠️ Tech Stack
* **Language:** Python 3.12+
* **Framework:** Slack Bolt for Python (`slack_bolt`)
* **AWS SDK:** Boto3
* **Cloud Infrastructure:** AWS Lambda, Amazon EC2, AWS IAM

## 📋 Prerequisites
* An **AWS Account** with permissions to manage IAM Roles, Lambda functions, and EC2 instances.
* A pre-existing **AWS Security Group** configured to allow SSH access.
* A **Slack Workspace** with permissions to create and install Slack Apps.

## ⚙️ Setup & Deployment Guide

### 1. Create the Slack App
1. Go to [api.slack.com/apps](https://api.slack.com/apps) and create a new app "From scratch."
2. Under **OAuth & Permissions**, add the following **Bot Token Scopes**:
   * `commands` (To listen for `/ec2`)
   * `chat:write` (To send success/error messages)
   * `files:write` (To upload the private key file)
   * `im:write` (To open direct message channels with users)
3. Install the app to your workspace and save your **Bot User OAuth Token** (`xoxb-...`) and **Signing Secret** (found under Basic Information).

### 2. Configure AWS IAM
1. In the AWS Console, create a new IAM Role for your Lambda function.
2. Attach the following policies:
   * `AWSLambdaBasicExecutionRole` (For CloudWatch logging)
   * `AmazonEC2FullAccess` (For provisioning servers and keys)
   * `AWSLambdaRole` (To allow the Lambda function to invoke itself for background processing).

### 3. Download the Deployment Package
AWS Lambda requires external libraries (like `slack_bolt`) to be bundled with the code. 
* You do **not** need to build this from scratch. Simply download the `.zip` file provided in this repository, which already contains the Python code and all necessary dependencies.

### 4. Deploy to AWS Lambda
1. Create a new AWS Lambda function (Python 3.x runtime) and assign the IAM Role created in Step 2.
2. Under the **Code** tab, click **Upload from > .zip file** and upload the `.zip` file you downloaded from this repo.
3. Under **Configuration > Environment variables**, add:
   * `SLACK_BOT_TOKEN`: Your `xoxb-` token.
   * `SLACK_SIGNING_SECRET`: Your Slack signing secret.
4. Under **Configuration > Function URL**, create a new Function URL with Auth Type `NONE`. Copy this URL.

### 5. Link Slack to Lambda
1. Go back to your Slack App Dashboard.
2. Under **Slash Commands**, create a new command called `/ec2` and paste your Lambda Function URL as the Request URL.
3. Under **Interactivity & Shortcuts**, toggle it **On** and paste the exact same Lambda Function URL.
4. Reinstall the app to your workspace if prompted (Slack requires this whenever you change scopes or add interactivity).

## 💻 Usage
1. Open any channel in your Slack workspace.
2. Type `/ec2` and press Enter.
3. Fill out the interactive modal with your desired Instance Name, OS, Type, and Storage Size.
4. Click **Launch Instance**.
5. Wait a few seconds, and you will receive a Direct Message from the bot containing your server's Public IP, SSH command, and the attached `.pem` private key!

## ⚠️ Important Notes
* **Cost:** Running `t2.micro` or `t3.micro` instances may fall under the AWS Free Tier, but leaving instances running will eventually incur charges. Remember to terminate instances from your AWS Console when you are done.
* **Security:** Passing private keys over chat is acceptable for portfolio projects and internal testing, but should be carefully evaluated against your organization's security policies before deploying to enterprise production environments.
