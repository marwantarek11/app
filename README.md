# Jenkins Openshift Deployment Automation

This project aims to automate the deployment process of an application to an OpenShift cluster using Jenkins. It utilizes a Jenkins slave to handle the deployment process. The steps involve building and pushing a Docker image to the OpenShift cluster, and then deploying the application using token credentials obtained from a service account. Additionally, it utilizes a shared library for reusability and easier maintenance.

## Prerequisites

- Jenkins master configured with this plugins
- Jenkins slave running on an EC2 instance
- OpenShift cluster access
- Docker installed on Jenkins slave
- GitHub repositories:
  - Application code: [marwantarek11/app](https://github.com/marwantarek11/app)
  - Shared Library: [marwantarek11/Shared-Library-App](https://github.com/marwantarek11/Shared-Library-App)

## Pipeline Stages
- Test
- Build Docker Image
- Editing Deployment.yml
- Push Docker Image
- Deploy Image On Openshift

  
## Setup

1. **Set Up GitHub Shared Library**:
   [marwantarek11/Shared-Library-App](https://github.com/marwantarek11/Shared-Library-App)

    1- runUnitTests.groovy
    ```groovy
    #!/usr/bin/env groovy
    def call() {
    	echo "Running Unit Test..."
    	sh './gradlew clean test'	
    }
    ```
    2- buildandPushDockerImage function
    ```groovy
    #!usr/bin/env groovy
    def call(String dockerHubCredentialsID, String imageName) {
    
    	// Log in to DockerHub 
    	withCredentials([usernamePassword(credentialsId: "${dockerHubCredentialsID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
    		sh "docker login -u ${USERNAME} -p ${PASSWORD}"
        }
        
        // Build and push Docker image
        echo "Building and Pushing Docker image..."
        sh "docker build -t ${imageName}:${BUILD_NUMBER} ."
        sh "docker push ${imageName}:${BUILD_NUMBER}"	 
    }
    ```
    3- EditNewImage
    ```groovy
    #!/usr/bin/env groovy

    def call(String imageName) {
    
    // Edit deployment.yaml with new Docker Hub image
    sh "sed -i 's|image:.*|image: ${imageName}:${BUILD_NUMBER}|g' deployment.yml"

    }
    ```
    4- Deploy Image On Openshift
    ```groovy
    #!/usr/bin/env groovy

    def call(String openshiftCredentialsID, String nameSpace, String clusterUrl) {

    
    // Login to OpenShift using the service account token
    withCredentials([string(credentialsId: openshiftCredentialsID, variable: 'OC_TOKEN')]) {
        sh "oc login --token=$OC_TOKEN --server=$clusterUrl --insecure-skip-tls-verify"
    }

    // Apply the updated deployment.yaml to the OpenShift cluster
    sh "oc apply -f . --namespace=${nameSpace}"
    }
    ```

  2. **Configure Jenkins System**
       ![Screenshot 2024-06-06 050350](https://github.com/marwantarek11/app/assets/167176241/f0b745fd-e73e-4b6d-8487-eeb5ab209268)

  3. **Configure Jenkins Slave**: Set up a Jenkins slave on an EC2 instance, ensuring Docker is installed on it.

     
      - Create Ec2 instance and make sure Instance type t3.medium for instance stability and create keypair to connect via ssh
    
      - Edit Network setting and apply VPC and Public subnet , Enable Auto-assign public IP, Create Security group assign Inbound Security Group Rules Type SSH port 22 , Add Security Group Rule type Custom TCP Port 8080 for Jenkins server.
    
      - Launch an instance
     
      - ![ec2](https://github.com/marwantarek11/app/assets/167176241/6c6659d9-a832-42cf-9cb0-ab9876413afa)

      -  Install Openshift,Docker,JDK
        ![oc-java-docker](https://github.com/marwantarek11/app/assets/167176241/fb115690-5256-466f-a0e1-770dbb3c8f68)

    


5. **Configure Jenkins Credentials**: Add credentials for accessing the OpenShift cluster. This can be done by creating a new OpenShift token credential in Jenkins credentials.
   ![cred](https://github.com/marwantarek11/app/assets/167176241/2a981b7f-fb7e-417c-91f7-e276174f3e0a)
   ![ubuntu-cred](https://github.com/marwantarek11/app/assets/167176241/a770b720-b6c1-4482-a1ad-a4d1baeda07f)


6. **Configure Jenkins Slave**: Set up a Jenkins Slave Configuration
     ![node-config1](https://github.com/marwantarek11/app/assets/167176241/21b870b2-45d3-454e-a6c9-6e9030327c66)
     ![node-config2](https://github.com/marwantarek11/app/assets/167176241/78ea9997-4cb0-4a34-a8f1-91e1f9e1571c)

    - Verify Slave Configured Successful:
        ![slave-verify](https://github.com/marwantarek11/app/assets/167176241/0d77699e-4b33-41b7-b631-b9f200d9ea21)
        ![slave-verfiy-2](https://github.com/marwantarek11/app/assets/167176241/0aa169b1-455d-44e7-878c-6c3f2e5a44cf)


7. **Configure Jenkins Pipeline**: Set up a Jenkins pipeline job that fetches the code from the application repository, builds the Docker image, pushes it to the OpenShift registry, and deploys the application to the OpenShift cluster using the shared library.
        ![jenkins-config1](https://github.com/marwantarek11/app/assets/167176241/229cc42f-b5df-4737-8703-751933149bcc)
        ![jenkins-config2](https://github.com/marwantarek11/app/assets/167176241/e833bb83-4938-4579-8b78-a807adac2ee3)
        

8. **Jenkins Pipeline Stages**:
      ![jenkins-build](https://github.com/marwantarek11/app/assets/167176241/2fd8d88a-5ded-4173-99be-62bbff7dbf7a)
      ![oc-get-all](https://github.com/marwantarek11/app/assets/167176241/b804bdee-fd6c-4315-9170-5321f039e8c9)
      ![jenkins-openshift](https://github.com/marwantarek11/app/assets/167176241/e5174021-0750-4bdd-ab86-19e005548b35)

