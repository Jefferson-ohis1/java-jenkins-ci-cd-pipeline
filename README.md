# Java Application CI/CD Pipeline with Jenkins and Docker

## Project Overview
This project demonstrates the implementation of a complete CI/CD pipeline for a Java application using Jenkins, Maven, Docker, GitHub, and AWS EC2.
The objective of this project was to automate the build and containerization process of a Java application. Jenkins was configured to pull source code from GitHub, build the application using Maven, create a Docker image, and push the image to Docker Hub automatically.

## Architecture
GitHub Repository → Jenkins Pipeline → Maven Build → Docker Image Build → Docker Hub

## The pipeline automatically:
- Pulls source code from GitHub
- Builds the Java application using Maven
- Creates a Docker image
- Pushes the Docker image to Docker Hub
The entire CI/CD process is managed through Jenkins running on an AWS EC2 instance.

## Technologies Used:
- Java 17
- Maven
- Jenkins
- Docker
- GitHub
- AWS EC2 (Ubuntu)
- Git Bash
- Visual Studio Code

## Project Structure: java-jenkins-ci-cd-pipeline/
- src/
- pom.xml
- Dockerfile
- Jenkinsfile
- README.md

## Step 1: Clone the Java Application Repository
- Clone the Java application repository using Git Bash
- Navigate into the project directory
- Open the project in Visual Studio Code: code .

## Step 2: Create Dockerfile: Created a Dockerfile to containerize the Java application.

# Build stage:
```
FROM maven:3.8.4-openjdk-17-slim AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests
```
# Run stage
```
FROM eclipse-temurin:17-jdk-jammy
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Dockerfile Explanation:
- Uses Maven and OpenJDK 17 to build the application.
- Packages the application into a JAR file.
- Uses a lightweight runtime image.
- Exposes port 8081.
- Runs the application using Java.

## Step 3: Launch Jenkins Server on AWS EC2: Create an AWS EC2 instance to host Jenkins.
- Security Group Configuration
  - SSH PORT: 22
  - HTTP PORT: 80
  - CUSTOM TCP(JENKINS): 8080

## Step 4: Connect to Jenkins Server
- SSH into the EC2 instance using the .PEM key

## Step 5: Install Java and Jenkins
- Created a script file: nano script.sh
- Added the following commands:
```
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins
```

- Run the script: sh script.sh

## Step 6: Verify Jenkins Installation
- Check Jenkins version: jenkins --version
- Verify Jenkins is listening on port 8080: sudo ss -tulpn | grep 8080

## Step 7: Access Jenkins Web Interface
- Open Jenkins in a browser: http://public-ipv4 address from aws running ec2:8080
- Retrieve the administrator password. From inside Jenkins server in gitbash: sudo cat /var/lib/jenkins/secrets/initialAdminPassword
- Paste the generated password into the Jenkins setup page.

## Step 8: Configure Jenkins: Install Suggested Plugins
- Selected: Install Suggested Plugins
- Create Admin User: Create the first Jenkins administrator account.

## Step 9: Install Additional Jenkins Plugins: 
- Installed the following plugins:
 - GitHub Integration
 - Docker
 - Docker Pipeline
 - Maven Integration
 - SSH Agent

## Step 10: Configure Jenkins Tools
- Navigate to: Manage Jenkins → Tools
- Configured: JDK
 - Name: jdk
 - Install automatically: Enabled
- Maven:
 - Name: maven3
 - Install automatically: Enabled

 ## Step 11: Configure Docker Hub Credentials
- Add Docker Hub credentials inside Jenkins: Manage Jenkins → Credentials
- Stored:
 - Docker Hub Username
 - Docker Hub Password
- Credential ID: DOCKER_LOGIN

## Step 12: Install Docker on Jenkins Server
- Create script file: nano script.sh
- Added:
```
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

- Execute script.sh file: sh script.sh
- Verify Docker installation: docker --version

## Step 13: Create Jenkins Pipeline
- Created a Jenkinsfile in the project root.
```
pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Java Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jeffersonohis1/my-java-app:v1 .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'DOCKER_LOGIN',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    )
                ]) {
                    sh '''
                        echo $PASSWORD | docker login -u $USERNAME --password-stdin
                        docker push jeffersonohis1/my-java-app:v1
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and Docker image push successful'
        }

        failure {
            echo '❌ Build failed. Check the logs for details'
        }
    }
}

```

## Step 14: Push Project to GitHub
- Initialize Git: git init
- Add project files: git add .
- Commit changes: git commit -m "Initial commit: Java application with Jenkins, Maven, and Docker"
- Create a GitHub repository: java-jenkins-ci-cd-pipeline
- Add remote: git remote set-url origin https://github.com/Jefferson-ohis1/java-jenkins-ci-cd-pipeline.git
- Push code: git push -u origin main

## Step 15: Create Jenkins Pipeline Job
- Navigate to: Jenkins Dashboard → New Item
- Job Name: java-docker-pipeline
- Select: Pipeline
- Configure:
 - Definition: Pipeline script from SCM
 - SCM: Git
 - Repository URL: https://github.com/Jefferson-ohis1/java-jenkins-ci-cd-pipeline.git
 - Branch: */main
- Save the configuration.

## Step 16: Build the Pipeline
- Click: Build Now

# Issue Encountered: Java Version Conflict
- Problem: The pipeline failed during execution because:
 - Jenkins server had Java 21 installed.
 - The Java application was developed and built using Java 17.
 - Jenkinsfile was configured to use JDK 17.
- This version mismatch caused the build process to fail.

# Resolution
## Installed Java 17 on the Jenkins server.
- Created a script.sh file in Jenkins server:
```
sudo apt update
sudo apt install -y openjdk-17-jdk
```
- Executed script.sh file: sh script.sh

- Verified installed JDK versions: ls /usr/lib/jvm/
- Output:
 - java-1.17.0-openjdk-amd64
 - java-1.21.0-openjdk-amd64
 - java-17-openjdk-amd64
 - java-21-openjdk-amd64
 - openjdk-17

- Restarted Jenkins: sudo systemctl restart jenkins
- Configured Jenkins to use Java 17: Manage Jenkins → Tools
 - Name: jdk17
 - JAVA_HOME: /usr/lib/jvm/java-17-openjdk-amd64
 - Install automatically: Disabled
- Saved the configuration.

# Final Result
- After configuring Jenkins to use JDK 17, the pipeline executed successfully.
- Pipeline stages completed successfully:
 - Source Code Checkout
 - Maven Build
 - Docker Image Build
 - Docker Hub Authentication
 - Docker Image Push
- Result:
 - Finished: SUCCESS

# Key Skills Demonstrated
- Jenkins CI/CD Pipeline Configuration
- AWS EC2 Administration
- Java Application Build Automation
- Maven Build Management
- Docker Containerization
- Docker Hub Integration
- Jenkins Credentials Management
- Git and GitHub Workflow
- Linux Server Administration
- Troubleshooting Build Failures
- JDK Version Management

# Author
# Jefferson Ohis
# DevOps Engineer | AWS Certified Cloud Practitioner
## GitHub: https://github.com/Jefferson-ohis1




