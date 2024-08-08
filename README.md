Let's go through the complete setup of AWS CodePipeline for deploying a basic HTML page from GitHub to an EC2 instance using AWS CodeDeploy.

### Prerequisites

1. **GitHub Repository:**
   - Create a GitHub repository (e.g., `simple-html-site`).
   - Add a basic `index.html` file:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Simple HTML Page</title>
   </head>
   <body>
       <h1>Welcome to My Simple HTML Page</h1>
   </body>
   </html>
   ```

   - Commit and push this file to your GitHub repository.

2. **Launch an EC2 Instance and install CodeDeploy Agent on EC2 :**
   - Launch an Amazon Linux 2 EC2 instance.
   - Ensure it has a public IP and can be accessed via SSH.
   - SSH into your EC2 instance.

   - Run the following commands to install and start the CodeDeploy agent:

   ```bash
   sudo yum update -y
   sudo yum install ruby -y
   cd /home/ec2-user
   wget https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install
   chmod +x ./install
   sudo ./install auto
   sudo service codedeploy-agent start
   ```

   - Verify that the CodeDeploy agent is running:

   ```bash
   sudo service codedeploy-agent status
   ```

4. **Create an S3 Bucket (Optional):**
   - If needed, create an S3 bucket to store the deployment artifacts (you can skip this step as CodePipeline will handle artifacts internally).

5. **Create an IAM Role for EC2:**
   - Create an IAM role with the following policies: `AmazonEC2RoleforAWSCodeDeploy` and `AmazonS3FullAccess`.
   - Attach this role to your EC2 instance.

6. **Create a CodeDeploy Application and Deployment Group:**
   - Go to the AWS CodeDeploy console.
   - Create a new application (e.g., `SimpleHTMLApp`).
   - Create a new deployment group (e.g., `SimpleHTMLDeploymentGroup`) with the following settings:
     - **Deployment type:** In-place.
     - **Environment configuration:** Amazon EC2 instances.
     - **Amazon EC2 tags:** Add a tag that matches your EC2 instance.
     - **Service role:** Use an IAM role with `AWSCodeDeployRole` permissions.

### Steps to Set Up CodePipeline

#### 1. Create the Pipeline

- Go to AWS CodePipeline in the AWS Management Console.
- Click "Create pipeline".
  
##### **Step 1: Pipeline Settings**
- **Pipeline name:** `SimpleHTMLPipeline`.
- **Role:** Select "New service role" to automatically create a role.
- **Artifact store:** Use the default location.

##### **Step 2: Add Source Stage**
- **Source provider:** GitHub.
- **Connection:** Connect your GitHub account.
- **Repository:** Select your repository (e.g., `simple-html-site`).
- **Branch:** `main` or `master`.
- **Change detection options:** AWS CodePipeline default.

##### **Step 3: Build Stage**
- **Skip build stage**: Since it's a simple HTML file, no build is needed.

##### **Step 4: Deploy Stage**
- **Deploy provider:** AWS CodeDeploy.
- **Application name:** `SimpleHTMLApp`.
- **Deployment group:** `SimpleHTMLDeploymentGroup`.

#### 2. Create the `appspec.yml` File

- In your GitHub repository, create a file named `appspec.yml` with the following content:

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  AfterInstall:
    - location: scripts/restart_server.sh
      timeout: 300
      runas: root
```

- Also, create the `scripts/restart_server.sh` file in your repository:

```bash
#!/bin/bash
sudo systemctl restart httpd
```

- Make sure to set the execute permissions on this script:

```bash
chmod +x scripts/restart_server.sh
```

- Commit and push these files to your GitHub repository.

#### 3. Configure EC2 to Serve HTML Page

- SSH into your EC2 instance and install Apache:

```bash
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```
#### 2. auto-trigger Finally 
- If you made any changes on code and push to git hub it triggers the code pipeline and new changes will be reflected

