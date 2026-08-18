# docker
Docker + Amazon ECR + GitHub Actions CI/CD
1. Project Overview
This project demonstrates a complete Docker-based CI/CD pipeline using GitHub Actions, Docker, Amazon ECR, and Amazon EC2.

Whenever code is pushed to the main branch:

GitHub Actions checks out the source code.
AWS credentials are configured.
GitHub Actions logs in to Amazon ECR.
A Docker image is built from the Dockerfile.
The image is pushed to Amazon ECR.
GitHub Actions connects to the EC2 server through SSH.
EC2 logs in to ECR.
EC2 pulls the latest Docker image.
The old container is stopped and removed.
A new container is started.
Old unused Docker images are cleaned up.
The running container is verified.
2. Architecture
                    Developer
                        |
                        | git push
                        v
                  GitHub Repository
                        |
                        v
                 GitHub Actions
                        |
              +---------+---------+
              |                   |
          Docker Build        AWS Login
              |                   |
              +---------+---------+
                        |
                        v
                Amazon ECR
                img1:latest
                        |
                        | SSH
                        v
                 Amazon EC2
                        |
                 docker pull
                        |
                        v
                 Docker Container
                    img1
                        |
                   5001:5000
                        |
                        v
                Flask Application
3. Technologies Used
Git & GitHub
GitHub Actions
Docker
Amazon ECR
Amazon EC2
AWS CLI
SSH
Flask/Python application
4. Dockerfile
The Dockerfile defines how the application image is created.

Example:

FROM ubuntu

WORKDIR /nit

COPY . /nit

RUN apt-get update && apt-get install -y python3 python3-pip && pip3 install --no-cache-dir --break-system-packages -r requirements.txt

EXPOSE 5000

CMD ["python3", "app.py"]
Dockerfile Instructions
FROM
FROM ubuntu
Selects Ubuntu as the base image.

WORKDIR
WORKDIR /nit
Creates/selects /nit as the working directory inside the container.

COPY
COPY . /nit
Copies application files from the Docker build context into /nit.

RUN
RUN ...
Executes commands during image creation.

In this project it:

Updates Ubuntu packages.
Installs Python3.
Installs pip.
Installs Python dependencies from requirements.txt.
EXPOSE
EXPOSE 5000
Documents that the Flask application listens on port 5000 inside the container.

CMD
CMD ["python3", "app.py"]
Starts the Flask application when the container starts.

5. Docker Installation on Amazon Linux
Install Docker:

sudo yum install docker -y
Start Docker:

sudo systemctl start docker
Enable Docker at boot:

sudo systemctl enable docker
Check Docker:

docker --version
Check Docker service:

sudo systemctl status docker
6. Build Docker Image
From the directory containing the Dockerfile:

docker build -t img1 .
Explanation:

docker build       -> Build Docker image
-t img1            -> Name the image img1
.                  -> Current directory is the build context
Check images:

docker images
7. Run Docker Container
Run the image:

docker run -dt --name img1 -p 5001:5000 img1
Port mapping:

EC2 Host Port       Container Port
     5001     --->       5000
                         |
                         v
                    Flask App
Check running containers:

docker ps
Check all containers:

docker ps -a
8. Useful Docker Commands
View images
docker images
View running containers
docker ps
View all containers
docker ps -a
View application logs
docker logs img1
Follow logs:

docker logs -f img1
Enter the container
docker exec -it img1 /bin/bash
Inside the container:

pwd
ls
python3 --version
pip3 --version
Exit:

exit
Stop container
docker stop img1
Start container
docker start img1
Remove container
docker rm img1
Remove image
docker rmi img1
View image layers
docker history img1
View container details
docker inspect img1
Check Docker disk usage
docker system df
Remove unused images
docker image prune -f
9. Push Docker Image to Amazon ECR
The Docker image is tagged with the ECR repository:

210948569624.dkr.ecr.us-east-1.amazonaws.com/img1:latest
Login to ECR:

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 210948569624.dkr.ecr.us-east-1.amazonaws.com
Tag the image:

docker tag img1:latest 210948569624.dkr.ecr.us-east-1.amazonaws.com/img1:latest
Push:

docker push 210948569624.dkr.ecr.us-east-1.amazonaws.com/img1:latest
Verify the image in the AWS ECR repository.

10. Pull Image from ECR
On the EC2 server:

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 210948569624.dkr.ecr.us-east-1.amazonaws.com
Pull:

docker pull 210948569624.dkr.ecr.us-east-1.amazonaws.com/img1:latest
Run:

docker run -d \
  --name img1 \
  --restart unless-stopped \
  -p 5001:5000 \
  210948569624.dkr.ecr.us-east-1.amazonaws.com/img1:latest
11. GitHub Actions CI/CD
The GitHub Actions workflow is triggered when code is pushed to main.

on:
  push:
    branches:
      - main
CI Process
GitHub Actions:

Checkout Code
     ↓
Configure AWS Credentials
     ↓
Login to ECR
     ↓
Docker Build
     ↓
Docker Push
     ↓
ECR
CD Process
After the image is pushed to ECR:

GitHub Actions
     ↓
SSH into EC2
     ↓
Login to ECR
     ↓
docker pull
     ↓
Stop old container
     ↓
Remove old container
     ↓
Run new container
     ↓
Verify deployment
12. GitHub Secrets
The workflow uses GitHub Secrets instead of directly writing sensitive credentials in the workflow.

Required secrets:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
EC2_SSH_KEY
These are referenced using:

${{ secrets.AWS_ACCESS_KEY_ID }}
${{ secrets.AWS_SECRET_ACCESS_KEY }}
${{ secrets.EC2_SSH_KEY }}
13. Deployment Commands Used by GitHub Actions
Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ECR_REGISTRY
Pull latest image
docker pull ECR_REGISTRY/img1:latest
Stop old container
docker stop img1 || true
Remove old container
docker rm img1 || true
Start new container
docker run -d \
  --name img1 \
  --restart unless-stopped \
  -p 5001:5000 \
  ECR_REGISTRY/img1:latest
Cleanup
docker image prune -f
Verify
docker ps
14. Final CI/CD Flow
Developer modifies application
             |
             v
        git add .
             |
             v
        git commit
             |
             v
        git push origin main
             |
             v
       GitHub Actions
             |
             v
       Checkout Code
             |
             v
        Docker Build
             |
             v
        Docker Image
             |
             v
          ECR Push
             |
             v
       Amazon ECR
             |
             v
        SSH to EC2
             |
             v
       ECR Login
             |
             v
        Docker Pull
             |
             v
    Stop Old Container
             |
             v
    Remove Old Container
             |
             v
      Run New Container
             |
             v
      Application Live
15. What I Learned
Through this task, I practiced:

Creating a Dockerfile.
Understanding Dockerfile instructions.
Building Docker images.
Running containers.
Mapping host and container ports.
Executing commands inside containers.
Checking Docker logs.
Managing Docker containers and images.
Creating and using an Amazon ECR repository.
Pushing Docker images to ECR.
Pulling Docker images from ECR.
Configuring GitHub Actions.
Automating Docker image builds.
Automating ECR image pushes.
Deploying Docker containers to EC2 through SSH.
Implementing a basic CI/CD pipeline.
Final Result
A code push to the main branch automatically triggers:

GitHub → Docker Build → ECR Push → EC2 Pull → Container Deployment

This demonstrates a basic real-world Docker CI/CD deployment using GitHub Actions, Amazon ECR, and Amazon EC2.