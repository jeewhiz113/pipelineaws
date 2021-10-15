-------
Instructions on creating a simple pipeline in AWS for CI/CD.

-----
Step 1: Create a CodeCommit Reporsitory.
  - This folder already has a sample application to be deployed.
  - Push this up to the codecommit

Step 2: Create an EC2 instance and install the CodeDeploy agent
  - We create the EC2 instance and install the cCodeDeploy agent ON the EC2 instance.  The Agent is a software package that enables an instance to be used in CodeDeploy deployment.  We also attach an IAM role to the instance (known as an instance role) to allow it to fetch files that the CodeDeploy agent uses to deploy your application.  (So the instance fetches the files that the agent uses to deploy my application)

  - Create a role in IAM, called it EC2InstanceRole and attach with it AMAZONEC2RoleforAWSCodeDeploy, again this role is attached to an instance and it allows EC2 to call AWS services on my behalf.

  - Launch an EC2 instnace.  Choose Amazon Machine Image (AMI), locate Amazon Linux 2 AMI (HVM), SSD Volume Type 
  - Choose an instance type (t2.micro free tier), then Configure Instance Details:
  Choose 1 instance, enable auto-assign public IP, pick previously created Role for IAM role
  - Expand Advanced Details, and in the User data field, enter the following:
  #!/bin/bash
  yum -y update
  yum install -y ruby
  yum install -y aws-cli
  cd /home/ec2-user
  wget https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install
  chmod +x ./install
  ./install auto

  -The above code installs the CodeDeploy agent on the instance as it is created.
  -Then click Add Storage, then Add Tags
  -Add the following as tags - Name MyCodePipelineDemo for key and value respectively.  CodeDeploy later selects instances to deploy based on the tags that are attached to instances.

Step 3: Create an application in CodeDeploy

In CodeDeploy, an application is a resource that contains the software application you want to deploy.  So the CodeDeploy application is a layer of software on top of the app I wish to deploy.  We later use this CodeDeploy application to automate deployments of the same application to the EC2 instance.

  - First we create a role that allows CodeDeploy to perform deployments.  Then we create a CodeDeploy Application.

  - Go to IAM -> Roles -> Create role -> use case: CodeDeploy -> Select: CodeDeploy -> Permissions
  - We should have AWSCodeDeployRole managed policy should already be attached to the role.
  - Next: Tags -> Next: Review -> Enter a name for the role: CodeDeployRole and then choose Create Role.

  - Go to CodeDeploy console -> Application -> Create Application -> Enter Application Name -> Choose EC2 Platform -> Create Application.

  -Next create a deployment group: a deployment group is a resource that defines deployment - related settings like which instances to deploy to and how fast to deploy them.

  -Create Deployment group -> enter MyDemoDeploymentGroup for name -> Choose CodeDeployRole for Service Role -> Choose In-place for deploymen type -> Environment Config: Amazon EC2 Instances, in the key field, enter Name, and value field enter MyCodePipeLineDemo -> Deployment Config: CodeDeployDefault.OneAtaTime -> Disable load balancer -> ignore Alarm configs under Advanced and click Create Deployment Group.

------
Next we create the CodePipeline

  - Go to the CodePipeline console
  - Create pipeline -> Enter MyFirstPipeline as the pipeline name -> Choose New Service Role -> Next
  - Add source -> pick the repo from CodeCommit and the Deployment Branch
  - Check CloudWatch events and CodePipeline Default -> Next
  - Skip build stage
  - Add deploy stage -> Choose AWS CodeDeploy in Deploy Provider -> Name: MyDemoApplication -> Deployment Group: MyDemoDeploymentGroup -> Create Pipeline

A few things of note: 

To verify that your pipeline ran successfully

View the initial progress of the pipeline. The status of each stage changes from No executions yet to In Progress, and then to either Succeeded or Failed. The pipeline should complete the first run within a few minutes.

After Succeeded is displayed for the pipeline status, in the status area for the Deploy stage, choose AWS CodeDeploy. This opens the CodeDeploy console. If Succeeded is not displayed see Troubleshooting CodePipeline.

On the Deployments tab, choose the deployment ID. On the page for the deployment, under Deployment lifecycle events, choose the instance ID. This opens the EC2 console.

On the Description tab, in Public DNS, copy the address (for example, ec2-192-0-2-1.us-west-2.compute.amazonaws.com), and then paste it into the address bar of your web browser.

This is the sample application you downloaded and pushed to your CodeCommit repository.