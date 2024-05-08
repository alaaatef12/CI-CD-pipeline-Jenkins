# Automated Build and Test with Jenkins on EC2 Instance
This project focuses on configuring Jenkins to automate the building and testing of code changes sourced from a GitHub repository, using a Jenkinsfile. The development process adheres to the Git Flow branching model.
## Table of Contents
1. [Create an EC2 Instance on AWS](#create-an-ec2-on-aws)
2. [prepare GitHub Repository](#prepare-github-repository)
3. [Install Git and Git Flow on EC2](#install-git-and-git-flow-on-ec2)
4. [Create a Jenkinsfile and Push it to GitHub](#create-a-jenkinsfile-and-push-it-to-github)
5. [Create a Jenkins Container on EC2](#create-a-jenkins-container-on-ec2)
6. [Configure Jenkins for GitHub Integration](#configure-jenkins-for-github-integration)
7. [Create a New Pipeline Job in Jenkins](#create-a-new-pipeline-job-in-jenkins)
8. [Implement Webhooks for CI](#implement-webhooks-for-ci)
9. [Testing and Validation](#Testing-and-Validation)
## 1. Create an EC2 Instance on AWS 
- Edit security group settings
  - Add inbound rule to the security group for port 8080.
    - type: custom TCP
    - port :8080
    - source: 0.0.0.0/0
## 2. Prepare GitHub Repository
- Create a new GitHub repository or select an existing one.
- Initialize the repository with a README.md
## 3. Install Git and Git Flow on EC2
    # install git 
      $ sudo yum install git
      $ git --version
      $ git clone "repo_url"    
      $ cd "path/to/the/repo"
      
    # initialize `develop` branch
      $ git checkout -b develop
      $ git push --set-upstream origin develop

    # install git flow
    $ git clone --recursive https://github.com/petervanderdoes/gitflow-avh.git
    $ cd gitflow-avh/contrib/
    $ chmod +x gitflow-installer.sh
    $ sudo ./gitflow-installer.sh install stable
    $ sudo yum update
    $ git flow init
    $ git branch
  ## 4. Create a Jenkinsfile and Push it to GitHub
- Create a Jenkinsfile:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```
- Save and exit.
- Push the file to the GitHub Repostory:
```bash
$ git add Jenkinsfile
$ git commit -m "Add Jenkinsfile"
$ git push "repo_url" develop
```
## 5. Create a Jenkins Container on EC2
```bash
$ sudo yum update -y
$ sudo yum install docker -y
$ sudo systemctl start docker
$ sudo docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```
- Accessing Jenkins:
    - In your web browser, go to: `http://(ec2-public-ip):8080`
    - Retrieve the Jenkins password:
        1. Run the following command inside your container:
        ```bash
        $ sudo docker exec -it "container id" bash
        $ cat /var/jenkins_home/secrets/initialAdminPassword
        ```
   - Copy the password to the Jenkins tab and sign in.
## 6. Configure Jenkins for GitHub Integration
- Install necessary plugins in Jenkins such as Git and Pipeline.
- Connect Jenkins to GitHub:
    - Go to "Manage Jenkins" > "Manage Plugins" > "Available" and install "GitHub Integration Plugin".
    - Set up credentials in Jenkins for GitHub (username and token).
## 7. Create a New Pipeline Job in Jenkins
- Select "New Item", name your pipeline (e.g., "github-pipeline"), and choose "Pipeline" as the type.
- In the pipeline configuration, select "Pipeline script from SCM" and choose "Git" as the SCM.
- Enter the repository URL and credentials.
- Specify the branch to build (e.g., */develop).
- In Build Triggers, check **"GitHub hook trigger for GITScm polling"**.
- Save.
## 8. Implement Webhooks for CI
- In GitHub, go to your repository settings and select **"Webhooks"**.
- **Add a new webhook:**
    - Payload URL: `http://<your-jenkins-url>:8080/github-webhook/`
    - Content type: **application/json**
    - Select **"Just the push event"**.
    - Ensure the webhook is active.
## 9. Testing and Validation
- Push a change to the 'develop' branch and verify Jenkins triggers a build.
- Check the Jenkins dashboard for build status and output.
