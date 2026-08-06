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
- 

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



