# Graded Assignment : Project: Deploying and Scaling Web Application

## Objective


---

## Prerequisites
Make sure you have the following installed on your system:
1. AWS CLI
2. Python and pip

---

## Instructions

### 1. AWS Environment setup
1. Install AWS CLI for windows from official website
2. Verify the installation
    ```bash
        aws --version
    ```
3. Configure AWS CLI:
    Run the following command and enter your AWS Access Key, Secret Key, Region, and Output format. You can get Access Key & Secret Key from AWS IAM policies
    ```bash
        aws configure
    ```
4. Install Boto3
    ```bash
        pip install boto3
    ```
---

### 2. Code setup
1. Fork the repository https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices to https://github.com/SyamalaKadmi/SampleMERNwithMicroservices.git
2. Clone the repository
    ```bash
     git clone https://github.com/SyamalaKadmi/SampleMERNwithMicroservices.git
    ```
---

### 3. Containerize the MERN Application
1. Create .env files for helloService & ProfileService in backend
    - helloService
        ```
            PORT=3001
        ```
    - profileService
        ```
            PORT=3002
            MONGO_URL="specifyYourMongoURLHereWithDatabaseNameInTheEnd"
        ```
2. Create docker files for frontend and backend (helloService & profileService) respectively
    - helloservice [HelloServiceDockerfile](backend/helloService/Dockerfile)
    - profileservice [profileServiceDockerfile](backend/profileService/Dockerfile)
    - frontendservice [frontendDockerfile](frontend/Dockerfile)
3. Build the docker images
    ```bash
    cd ./SampleMERNwithMicroservices
    docker build -t mern-frontend-image ./frontend
    docker build -t mern-backend-helloservice-image ./backend/helloService
    docker build -t mern-backend-profileservice-image ./backend/profileservice
    ```
4. Push Docker Images to Amazon ECR:
    1. Create a reppository for each image
        ```bash
            aws ecr create-repository --repository-name mern-frontend-repo
            aws ecr create-repository --repository-name mern-backend-helloservice-repo
            aws ecr create-repository --repository-name mern-backend-profileservice-repo
        ```
        ![ecrRepoCreation](Images/ecrCreation.png)
    2. Authenticate Docker to ECR
        ```bash
            aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
        ```
    3. Tag and push the docker images to the ECR
        - Frontend
        ```bash
            docker tag mern-frontend-image:latest 975050024946.dkr.ecr.us-east-1.amazonaws.com/mern-frontend-repo:latest
            docker push 975050024946.dkr.ecr.us-east-1.amazonaws.com/mern-frontend-repo:latest
        ```
        ![FrontEndECR](Images/ecrFrontend.png)

        - backend - helloService
        ```bash
            docker tag mern-backend-helloservice-image:latest 975050024946.dkr.ecr.us-east-1.amazonaws.com/mern-backend-helloservice-repo:latest
            docker push 975050024946.dkr.ecr.us-east-1.amazonaws.com/mern-backend-helloservice-repo:latest
        ```
        ![helloserviceECR](Images/ecrHelloService.png)

        - backend - profileService
        ```bash
            docker tag mern-backend-profileservice-image:latest 975050024946.dkr.ecr.us-east-1.amazonaws.com/mern-backend-profileservice-repo:latest
            docker push 975050024946.dkr.ecr.us-east-1.amazonaws.com/mern-backend-profileservice-repo:latest
        ```
        ![profileServiceECR](Images/ecrProfileService.png)

---

### 4. Version Control
1. Create CodeCommit Repository
    ```bash
        aws codecommit create-repository --repository-name MERNWithMicroServices --repository-description "MERN with Microservices - frontend & backend with helloService & profileservice"
    ```
    Proceeding with Github as code repository due to permission issues
    ![codeCommitIssue](Images/codeCommitError.png)
2. Jenkins Setup & Configuration
    1. Setup a Jenkins server on ec2 instance
    2. GitHub Access Token: Create a personal access token with ```repo``` and ```admin:repo_hook``` permissions in Github
    3. Install required plugins on Jenkins Server - GitHub Integration, Docker Pipeline, Amazon ECR, Pipeline
    4. Configure Github credentials in Jenkins. 
        - Go to Jenkins Dashboard -> Manage Jenkins -> Credentials -> System -> Global credentials (unrestricted) -> Add Credentials
        - Add Kind: Secret text, Secret Github Personal Access Token, ID: github-token, Description: GitHub Token for Jenkins
3. Create a Jenkins Pipeline Job
    1. Go to Jenkins Dashboard -> New Item
    2. Enter the job name (e.g., MERN-CI-CD-Pipeline) and select Pipeline and Click Ok
    3. Add the Github url -- https://github.com/SyamalaKadmi/SampleMERNwithMicroservices.git
    4. Under Build Triggers: Check GitHub hook trigger for GITScm polling. This allows Jenkins to trigger builds on new commits.
    5. Add the Pipeline script --> JenkinsFile
4. Configure Webhook 
    1. In GitHub Repository, go to settings -> Webhooks -> Add Webhook -> Payload url 
        ```
            http://<EC2-Public-IP>:8080/github-webhook/
        ```
        Content type: Application/json
        Choose Just the push event and click Add webhook
5. Validation 
    1. Verify the Jenkins pipeline is running successfully after a commit is made to repository
        ![JenkinsSuccess](Images/JenkinsSuccess.png)
    2. Verify images to pushed to AWS ECR
        - Frontend
        ![Frontend](Images/FrontendJenkins.png)
        - backend/helloService
        ![HelloService](Images/helloServiceJenkins.png)
        - backend/profileService
        ![ProfileService](Images/profileServiceJenkins.png)

---

### 5. Infrastructure as Code (IaC) with Boto3
Create a python script [setup_infrastructure.py](setup_infrastructure.py) to 
    - create VPC with public & private subnets
    - setup security groups for frontend & backend
    Verification - 
    ![IaC](Images/IACwithBoto3.png)

### 6. Deploying Backend Services
1. Create a launch template with details of deployment of helloService & ProfileService for backend using python script - [create_launch_template.py](create_launch_template.py)
    ![LaunchTemplate](Images/LaunchTemplate.png)
2. Create ASG using [create_asg.py](create_asg.py)
Verification: 
    - Run the script and verify its successfully ran
    ```bash
        python3 create_asg.py
    ```
    ![ASG](Images/ASG.png)
    - Verify that its created in AWS console
    ![ASGImage](Images/ASGImage.png)
### 7. Set Up Networking
1. Create a load balancer for the backend ASG
    ![BackendLoadbalancer](Images/BackendLoadbalancer.png)
2. Configure DNS


## Troubleshooting
1. Error with connecting to ECR while running Jenkins Pipeline
[Pipeline] sh
+ docker login --username AWS --password-stdin 975050024946.dkr.ecr.us-east-1.amazonaws.com
+ aws ecr get-login-password --region us-east-1

Unable to locate credentials. You can configure credentials by running "aws configure".
Error: Cannot perform an interactive login from a non TTY device

Solution: This might be because aws cli is configured with ec2-user and Jenkins pipeline might be running as 'Jenkins'
1. After configuring aws cli, credentials are stored in the system. Copy them to the Jenkins user
    ```bash
        sudo mkdir -p /var/lib/jenkins/.aws
        sudo cp ~/.aws/credentials /var/lib/jenkins/.aws/credentials
        sudo cp ~/.aws/config /var/lib/jenkins/.aws/config
    ```
2. Ensure the jenkins user has permissions to read these files
    ```bash
        sudo chown -R jenkins:jenkins /var/lib/jenkins/.aws
        sudo chmod 600 /var/lib/jenkins/.aws/credentials
        sudo chmod 600 /var/lib/jenkins/.aws/config
    ```
3. Restart Jenkins
    ```bash
        sudo systemctl restart Jenkins
    ```
4. Retry the pipeline


    
