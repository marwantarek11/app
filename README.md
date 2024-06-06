# Jenkins Pipeline Project

This project sets up a Jenkins pipeline to automate the Continuous Integration and Continuous Deployment (CI/CD) process for an application. The pipeline includes stages for testing the application, building a Docker image, pushing the image to Docker Hub, and deploying the application on OpenShift. The pipeline leverages an EC2 instance as a Jenkins slave and uses a shared library hosted on GitHub for reusable pipeline code.

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Pipeline Stages](#pipeline-stages)
- [Setup Instructions](#setup-instructions)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Project Overview
This Jenkins pipeline project aims to streamline the CI/CD process by automating key tasks:
- **Testing**: Runs automated tests to ensure the application is functioning correctly.
- **Building**: Creates a Docker image of the application.
- **Pushing**: Uploads the Docker image to Docker Hub.
- **Deploying**: Deploys the application on OpenShift.

The project utilizes an EC2 instance as a Jenkins slave to distribute the workload and a shared library from GitHub to incorporate reusable pipeline code, ensuring modularity and maintainability.

## Prerequisites
- Jenkins installed and configured
- Docker installed on the Jenkins slave (EC2 instance)
- Docker Hub account
- OpenShift cluster
- GitHub repository for the shared library

## Pipeline Stages
1. **Test**: Executes unit tests to validate the application code.
2. **Build**: Builds a Docker image for the application.
3. **Push**: Pushes the Docker image to Docker Hub.
4. **Deploy**: Deploys the Docker image to an OpenShift cluster.

## Setup Instructions
1. **Setup Jenkins Slave**:
   - Launch an EC2 instance and configure it as a Jenkins slave.
   - Install Docker on the EC2 instance.

2. **Configure Jenkins**:
   - Install necessary Jenkins plugins (e.g., Docker Pipeline, GitHub, OpenShift Pipeline).
   - Add the EC2 instance as a Jenkins slave.

3. **Shared Library**:
   - Clone the shared library repository from GitHub.
   - Configure Jenkins to use the shared library.

4. **Create Jenkins Pipeline**:
   - Define the pipeline script (`Jenkinsfile`) in your application repository.
   - Include stages for testing, building, pushing, and deploying the application.

## Usage
To use this pipeline, follow these steps:
1. Commit your code changes to your application repository.
2. Push the changes to trigger the Jenkins pipeline.
3. Monitor the pipeline execution on the Jenkins dashboard.
4. Verify the deployment on OpenShift.

Example `Jenkinsfile`:
```groovy
@Library('shared-library') _

pipeline {
    agent { label 'ec2-slave' }
    
    stages {
        stage('Test') {
            steps {
                script {
                    runTests()
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    buildDockerImage()
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    pushDockerImage()
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    deployToOpenShift()
                }
            }
        }
    }
}
```


