
# Company Website - Jenkins CI/CD Project

## Project Overview

A responsive company landing page built using HTML5 and CSS3,
containerized using Docker and deployed using Jenkins CI/CD.

## Technologies Used

- AWS EC2
- Ubuntu Linux
- GitHub
- Jenkins
- Jenkins Agent
- Docker
- Docker Hub
- Nginx
- SSH
- HTML5
- CSS3

## Architecture

The project uses three separate servers:

1. Jenkins Server - Pipeline management.
2. Build Server - Docker image build and push.
3. Deployment Server - Docker image pull and application hosting.

## Project Structure

- index.html - Website content
- style.css - Website styling
- Dockerfile - Docker image configuration
- Jenkinsfile - CI/CD pipeline
- README.md - Project documentation

## CI/CD Pipeline

1. Checkout source code from GitHub.
2. Build Docker image on the build server.
3. Authenticate with Docker Hub.
4. Push Docker image to Docker Hub.
5. Connect to the deployment server using SSH.
6. Pull the latest Docker image.
7. Stop and remove the previous container.
8. Run the new container.
9. Verify the website deployment.

## Application Port

Host Port: 9090
Container Port: 80

## Access the Website

http://DEPLOYMENT-PUBLIC-IP:9090

## Security

Docker Hub tokens and SSH private keys are stored securely
using Jenkins Credentials.
