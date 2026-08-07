# AWS Services

This project is for the DevOps Bootcamp demo for:

AWS Services - [DevOps Bootcamp](https://techworld-with-nana.teachable.com/p/devops-bootcamp)

## Demo Project

- Deploy Web Application on EC2 Instance (manually)
- CD - Deploy Application from Jenkins Pipeline to EC2 Instance
- CD - Deploy Application from Jenkins Pipeline on EC2 Instance (automatically with docker-compose)
- Complete the CI/CD Pipeline (Docker-Compose, Dynamic versioning)
- Create repository on AWS and push to private Docker registry
- Interacting with AWS CLI

### Technologies used

AWS, Jenkins, Docker, Linux, Git, Java, Maven, Docker Hub, Amazon ECR

### Project Description

- Create and configure an EC2 Instance on AWS
- Install Docker on remote EC2 Instance
- Deploy Docker image from private Docker repository on EC2 Instance
- Prepare AWS EC2 Instance for deployment (Install Docker)
- Create ssh key credentials for EC2 server on Jenkins
- Extend the previous CI pipeline with deploy step to ssh into the remote EC2 instance and deploy newly built image from Jenkins server
- Configure security group on EC2 Instance to allow access to our web application
- Install Docker Compose on AWS EC2 Instance
- Create docker-compose.yml file that deploys our web application image
- Configure Jenkins pipeline to deploy newly built image using Docker Compose on EC2 server
- Improvement: Extract multiple Linux commands that are executed on remote server into a separate shell script and execute the script from Jenkinsfile
- CI step: Increment version
- CI step: Build artifact for Java Maven application
- CI step: Build and push Docker image to Docker Hub
- CD step: Deploy new application version with Docker Compose
- CD step: Commit the version update
- Create private Docker registry on AWS (Amazon ECR)
- Tag and Push Docker image to this private repository

### Implementation

#### Deploy Web Application on EC2 Instance (manually)

Login to AWS console & launch a new EC2 instance. Choose the appropriate image, create a new key pair & security group (allow port 3080). Install docker in the instance

```bash
# move the key to ssh direct & modify the permissions
chmod 400 ~/.ssh/key.pem
# ssh to newly created instance
ssh -i ~/.ssh/key.pem ec2-user@x.x.x.x
# install & run docker
sudo yum update
sudo yum install docker
sudo service docker start
ps aux | grep docker
```

To run docker commands without sudo, add the current user to docker group , restart the session for the changes to take effect.

```bash
sudo usermod -aG docker $USER
# list all groups current user is added to
groups
```

Build the docker image for the project in the repo "https://github.com/mustafa-saleh/demo-react-nodejs-example". Create a private registry in docker hub & push the built image to it. From the EC2 instance, login to docker hub for authentication & pull the created image.

```bash
docker login
docker run -d -p 3080:3080 <username>/demo-app:react-node-example-1.0
```

Navigate to the browser & check the app is now running on port 3080

![Deploy to EC2](./images/react_node_app.png)

#### CD - Deploy Application from Jenkins Pipeline to EC2 Instance

Create docker file to build the image 

```dockerfile
FROM amazoncorretto:17-alpine-jdk

EXPOSE 8080

COPY ./target/java-maven-app-1.1.0-SNAPSHOT.jar /usr/app/
WORKDIR /usr/app

ENTRYPOINT ["java", "-jar", "java-maven-app-1.1.0-SNAPSHOT.jar"]
```

[Module 8](https://github.com/mustafa-saleh/demo-module-8-build-automation-and-ci-cd-with-jenkins) contains instruction on how to setup Jenkins & create a shared library to build maven projects. Follow the instructions create & configure a Jenkins multibranch pipeline to build & deploy the java project.

Install the Jenkins plugin "SSH Agent" to be used in the pipeline deployment stage. Add (pipeline scoped) SSH credentials in Jenkins to connect to the AWS EC2 instance during the deployment. Select credentials type "SSH Username with Private Key", set the id "ec2-server-key", username "ec2-user" & paste the private key to create the credentials.

Below is the Jenkins file build the app, push the image to github repository & deploy it to AWS EC2. 

```groovy
library identifier: 'my-shared-library@main', retriever: modernSCM([
    $class: 'GitSCMSource',
    remote: 'https://github.com/mustafa-saleh/demo-module-8-jenkins-shared-library.git',
    credentialsId: 'github-repo'
])

def gv

pipeline {
    agent any

    tools {
        maven 'maven-3.9.16'
    }

    environment {
        IMAGE_NAME = 'mustafa199b/demo:java-maven-1.0'
    }

    stages {
        stage('build jar') {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage('build image') {
            steps {
                script {
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerCmd = "docker run -p 8080:8080 -d ${IMAGE_NAME}"
                    
                    // sshagent from jenkins plugin "ssh agent"
                    sshagent(['ec2-server-key']) {
                        // -o StrictHostKeyChecking=no used to suppress the SSH pop-up
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.81.125.74 ${dockerCmd}"
                    }
                }
            }
        }
    }
}
```

Configure the security group on AWS to allow access on port 8080 used by the application.

Navigate to the browser on port 8080 and check the app is running

![Jenkins EC2 Deployment](./images/deploy_jenkins_ec2.png)

#### CD - Deploy Application from Jenkins Pipeline on EC2 Instance (automatically with docker-compose)

Let's create a docker-compose file to start all the services required by the application. First install docker-compose on the EC2 instance

```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version
```

Below is "docker-compose.yaml" to run the application and postgres

```yaml
version: '3.8'
services:
    java-maven-app:
      image: mustafa199b/demo:java-maven-1.0
      ports:
        - 8080:8080
    postgres:
      image: postgres:15
      ports:
        - 5432:5432
      environment:
        - POSTGRES_PASSWORD=my-pwd
```

Stop the running containers on the EC2 instance and modify the deployment stage to use the docker-compose file

```groovy
...
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerComposeCmd = 'docker-compose --detach -f docker-compose.yaml up'
                    
                    // sshagent from jenkins plugin "ssh agent"
                    sshagent(['ec2-server-key']) {
                        // -o StrictHostKeyChecking=no used to suppress the SSH pop-up
                        sh "scp docker-compose.yaml ec2-user@3.81.125.74:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.81.125.74 ${dockerComposeCmd}"
                    }
                }
            }
        }
...
```

If you need to run extra commands (ex: set env vars), use shell script & run it on the EC2 server. create "server-cmds.sh" & copy it to server. To avoid hardcoding the image name in docker-compose file let's pass it as a param to the shell script from jenkinsfile & then export it as env variable in the EC2 instance.

```bash
#!/usr/bin/env bash

export IMAGE=$1
docker-compose -f docker-compose.yaml up --detach
echo "success"
```

Update the docker-compose file to read IMAGE environment variable

```yaml
version: '3.8'
services:
    java-maven-app:
      image: ${IMAGE}
...
```

We can also refactor the deployment stage to remove any duplicate syntax like below

```groovy
...
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def ec2Instance = "ec2-user@3.81.125.74"
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    
                    // sshagent from jenkins plugin "ssh agent"
                    sshagent(['ec2-server-key']) {
                        // -o StrictHostKeyChecking=no used to suppress the SSH pop-up
                        sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }
...
```

#### Complete the CI/CD Pipeline (Docker-Compose, Dynamic Versioning)

To implement dynamic versioning, refer to the section [Dynamically Increment Application Version in Jenkins Pipeline](https://github.com/mustafa-saleh/demo-module-8-build-automation-and-ci-cd-with-jenkins/blob/main/README.md#dynamically-increment-application-version-in-jenkins-pipeline) in Module 8 Jenkins CI/CD.

Let's add the new pipeline stages "increment version" & "commit version update" as below

```groovy
...
        stage('increment version') {
            steps {
                script {
                    echo "incrementing the version..."
                    // increment patch number
                    sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                    versions:commit'
                    // parse the updated file to get the new version and save it as IMAGE_NAME for following stages
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    echo "new version is: ${version}"
                    env.IMAGE_NAME = "mustafa199b/demo:java-maven-$version-$BUILD_NUMBER"
                }
            }
        }
...
...
        stage('commit version update') {
            steps {
                script {
                    echo "incrementing the version..."
                    withCredentials([gitUsernamePassword(credentialsId: 'github-pass-token', gitToolName: 'Default')]) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "Jenkins"'
                        
                        sh 'git add .'
                        sh "git commit -m \"ci: Increment version to ${IMAGE_NAME}\""
                        sh 'git push origin HEAD:main'
                    }
                }
            }
        }
...
```

Remove any hardcoded versions in the docker file, and use regular expressions to get the latest build

```dockerfile
FROM amazoncorretto:17-alpine-jdk

EXPOSE 8080

# COPY ./target/java-maven-app-1.1.0-SNAPSHOT.jar /usr/app/
COPY ./target/java-maven-app-*.jar /usr/app/
WORKDIR /usr/app

# ENTRYPOINT ["java", "-jar", "java-maven-app-1.1.0-SNAPSHOT.jar"]
CMD java -jar java-maven-app-*.jar
```

Delete the environment variable IMAGE hardcoded in the pipeline since it's now getting set in "increment version" stage

```groovy
...
    // environment {
    //     IMAGE_NAME = 'mustafa199b/demo:java-maven-1.0'
    // }
...
```

In github repo, create access token to grant Jenkins access to commit the version update to the repository & create the credentials "github-pass-token" in Jenkins using the token.

#### Create Repository on AWS and Push to Private Docker Registry

Best practice is to create a dedicated AWS ECR repository for each distinct container image or microservice application.

To work with AWS ECR, let's install the AWS CLI version 2. Installation instructions can be found in URL "https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html". AWS CLI can also be installed via homebrew for MAC Book devices

```bash
brew update
brew install awscli
aws --version
```

Once CLI is installed, authenticate to access your account 

```bash
# enter access key id, secret access key, region, format (json) when prompted. running this will create a directory in home ".aws" with "config" and "credentials" files
aws configure
```

Let's build a docker from the project [js-app](https://github.com/mustafa-saleh/demo-js-app) and push it to the registry.

```bash
docker build -t my-app:1.0 .
```

In AWS console, select the ECR service & create the new registry "my-app", click the "View Push Commands" button for instructions on how to tag the docker image and push it to the registry.

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
docker tag my-app:1.0 <account-id>.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0
```

Check that the image has been pushed to the registry
![AWS ECR](./images/aws_ecr.png)
