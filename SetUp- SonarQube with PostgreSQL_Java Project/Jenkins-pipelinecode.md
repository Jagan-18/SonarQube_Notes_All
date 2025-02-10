# Jenkins Pipeline for Boardgame Project

This repository contains the Jenkins pipeline configuration for automating the build, test, package, and SonarQube analysis for the **Boardgame** project.

## Overview

The Jenkins pipeline automates the following tasks:

1. **Git Checkout**: Retrieves the latest code from the `main` branch of the GitHub repository.
2. **Compilation**: Compiles the code using Maven.
3. **Testing**: Runs unit tests to ensure code quality and functionality.
4. **Packaging**: Packages the application into a deployable artifact (JAR, WAR, etc.).
5. **SonarQube Analysis**: Analyzes the code quality using SonarQube.
6. **Quality Gate Check**: Waits for SonarQube's quality gate and decides whether to proceed or abort based on the results.

---
Pipeline-Code
---
pipeline {
    agent any
     tools {
         maven 'maven3'
     }
     environment{
         SCANNER_HOME = tool 'sonar-scanner'
     }
     
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jagan-18/Boardgame.git'
            }
        }
        stage('compilation') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                         $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectkey=Boardgame -Dsonar.projectName=Boardgram \
                          -Dsonar.java.binaries=target  '''
                }
                
            }
        }
        
        stage('Quality Gate check ') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'token_sonar'
                }
            }
        }
    }
}

---

---
Here the  clear explanation of your Jenkins pipeline setup:
---
## Jenkins Pipeline Configuration

This pipeline is defined using Jenkins' **Declarative Pipeline** syntax and follows these stages:

### 1. `git checkout`
- **Purpose**: Pulls the latest code from the `main` branch.
- **Command**:
    ```groovy
    git branch: 'main', url: 'https://github.com/Jagan-18/Boardgame.git'
    ```

### 2. `compilation`
- **Purpose**: Compiles the source code using Maven.
- **Command**:
    ```groovy
    sh 'mvn compile'
    ```

### 3. `Tests`
- **Purpose**: Executes unit tests using Maven to ensure the correctness of the code.
- **Command**:
    ```groovy
    sh 'mvn test'
    ```

### 4. `Package`
- **Purpose**: Packages the application as a JAR, WAR, or other format using Maven.
- **Command**:
    ```groovy
    sh 'mvn package'
    ```

### 5. `SonarQube Analysis`
- **Purpose**: Runs SonarQube analysis to check code quality.
- **Setup**: The SonarQube environment variable (`SCANNER_HOME`) is configured using the tool `sonar-scanner`.
- **Command**:
    ```groovy
    withSonarQubeEnv('sonar-server') {
        sh '''
             $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectkey=Boardgame -Dsonar.projectName=Boardgram \
             -Dsonar.java.binaries=target
        '''
    }
    ```

### 6. `Quality Gate Check`
- **Purpose**: Waits for SonarQube's quality gate to pass. If the quality gate fails, the pipeline is aborted. It waits for a maximum of 1 hour for the quality gate results.
- **Command**:
    ```groovy
    timeout(time: 1, unit: 'HOURS') {
        waitForQualityGate abortPipeline: false, credentialsId: 'token_sonar'
    }
    ```

---

## Prerequisites

- **Jenkins** installed and configured.
- **SonarQube** server running with the project set up.
- **Maven** installed and configured with version `maven3`.
- **Sonar Scanner** installed and configured on Jenkins.

## Tools Required

1. **Maven**: Java build tool for compiling, testing, and packaging the application.
2. **SonarQube Scanner**: Used for code quality analysis.

Ensure that you have the necessary tools and credentials set up in your Jenkins environment.

---

## How to Use the Pipeline

1. **Clone the Repository**: Make sure to clone the `Boardgame` repository in Jenkins.

2. **Create a New Pipeline Job in Jenkins**:
    - In the Jenkins dashboard, create a new **Pipeline** job.
    - In the **Pipeline** section, choose **Pipeline script from SCM**.
    - Select **Git** and provide the repository URL: `https://github.com/Jagan-18/Boardgame.git`.
    - Set the branch to `main`.

3. **Configure SonarQube in Jenkins**:
    - Go to **Manage Jenkins** > **Configure System**.
    - Add a **SonarQube Server** and provide the necessary credentials and configuration.

4. **Run the Pipeline**: After configuring everything, you can trigger the Jenkins pipeline manually or on every commit to the `main` branch.

---

## Additional Information

- **Quality Gate**: The `Quality Gate` is a mechanism in SonarQube to ensure that the code quality meets the defined standards. If the quality gate fails, the pipeline can be configured to abort, ensuring that poor-quality code doesn't make it to production.
- **Timeout**: A 1-hour timeout is set to wait for the SonarQube quality gate to ensure that the pipeline doesn't hang indefinitely if the quality gate takes longer.

---

## Troubleshooting

- If the pipeline fails at the `SonarQube Analysis` step, ensure that:
  - SonarQube is up and running.
  - The SonarQube credentials are correctly configured in Jenkins.
  - The `sonar-scanner` tool is correctly installed and configured in Jenkins.
  
- If the `Quality Gate Check` step fails, review the SonarQube quality gate settings to ensure the project meets the required standards.
---
---