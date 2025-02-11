---

# **Jenkins Pipeline Setup for NodeJS with SonarQube Analysis**

This guide explains how to configure a Jenkins pipeline to automate the following tasks:

1. **Git Checkout**: Clone the repository.
2. **Install Dependencies**: Install the necessary dependencies using `npm`.
3. **Run Tests**: Execute tests to ensure code quality.
4. **SonarQube Analysis**: Perform static code analysis using SonarQube.

---

## **Prerequisites**

Before proceeding with the pipeline configuration, ensure you have the following:

- A **Jenkins Server** running with the necessary plugins (e.g., NodeJS, SonarQube).
- **SonarQube** server setup and configured (either locally or remotely).
- **Node.js** and **npm** installed on the Jenkins agents.
- A **GitHub** repository containing the Node.js application and tests.

---

## **Pipeline Overview**

The Jenkins pipeline consists of the following stages:

1. **Git Checkout**: Clones the repository from GitHub.
2. **Install Dependencies**: Installs required npm packages.
3. **Run Tests**: Executes tests via `npm run test`.
4. **SonarQube Analysis**: Analyzes the project using SonarQube.

---

## **Jenkinsfile: Detailed Explanation**

```groovy
pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner' // Reference to the SonarQube scanner tool
    }

    stages {
        // Stage 1: Git Checkout
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jagan-18/NodeJS-JEST.git' // Clones the repository
            }
        }

        // Stage 2: Install Dependencies
        stage('Dependencies') {
            steps {
                nodejs('nodejs20') {  // Using Node.js 20 version
                    sh 'npm install'  // Installs npm packages
                }
            }
        }

        // Stage 3: Run Tests
        stage('Tests') {
            steps {
                nodejs('nodejs20') {  // Using Node.js 20 version
                    sh 'npm run test'  // Runs tests using npm
                }
            }
        }

        // Stage 4: SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') { // Using the SonarQube server configuration
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=nodejstest \
                    -Dsonar.projectKey=nodejstest \
                    -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/*.test.js \
                    -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    '''
                }
            }
        }
    }
}
```

---

## **Pipeline Breakdown**

### **1. Git Checkout Stage**

This stage clones the GitHub repository where the Node.js application and tests are stored.

```groovy
git branch: 'main', url: 'https://github.com/Jagan-18/NodeJS-JEST.git'
```

- **Explanation**: It fetches the code from the `main` branch of the GitHub repository.
  
### **2. Install Dependencies Stage**

Here, we install the necessary npm dependencies using the `npm install` command.

```groovy
nodejs('nodejs20') {
    sh 'npm install'
}
```

- **Explanation**: 
  - The `nodejs('nodejs20')` block ensures that the pipeline uses Node.js version 20.
  - The `sh 'npm install'` command installs all dependencies defined in the `package.json` file.

### **3. Run Tests Stage**

This stage runs your test cases defined in the project using the `npm run test` command.

```groovy
nodejs('nodejs20') {
    sh 'npm run test'
}
```

- **Explanation**: 
  - Runs the tests for your project, typically defined in `package.json` under the `"scripts"` section (`"test": "jest"` or similar).
  - You can replace this with any other test framework command depending on the structure of your project.

### **4. SonarQube Analysis Stage**

SonarQube performs static code analysis on your project to detect code quality issues, security vulnerabilities, and maintainability problems.

```groovy
withSonarQubeEnv('sonar-server') {
    sh '''
    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=nodejstest \
    -Dsonar.projectKey=nodejstest \
    -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/*.test.js \
    -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
    '''
}
```

- **Explanation**:
  - `withSonarQubeEnv('sonar-server')`: This block ensures the SonarQube environment variables are loaded from Jenkins. The server name (`sonar-server`) should match the configuration in Jenkins.
  - `sonar-scanner`: This command initiates the analysis using the SonarQube scanner.
  - **SonarQube parameters**:
    - `-Dsonar.projectName=nodejstest`: The name of your SonarQube project.
    - `-Dsonar.projectKey=nodejstest`: The unique key for the project in SonarQube.
    - `-Dsonar.sources=.`: Specifies the source code directory (in this case, the root directory).
    - `-Dsonar.tests=.`: Specifies the test directory.
    - `-Dsonar.test.inclusions=**/*.test.js`: Tells SonarQube to include files with the `.test.js` extension for test coverage.
    - `-Dsonar.javascript.lcov.reportPaths=coverage/lcov.info`: Specifies the location of the coverage report, which SonarQube will use to measure test coverage.

---

## **SonarQube Configuration**

### **SonarQube Server Setup in Jenkins**

1. **Install SonarQube Scanner**:
   - You need to install the SonarQube Scanner in Jenkins, which is used to run the analysis.
   - To configure it, go to Jenkins → Manage Jenkins → Global Tool Configuration → SonarQube Scanner.

2. **Configure SonarQube Server**:
   - Under "SonarQube Servers", add the server configuration (URL, authentication token).
   - The server name used in the pipeline (`sonar-server`) must match the name configured in Jenkins.

---

## **Execution and Results**

Once you run this pipeline:

1. **Git Checkout**: The latest code will be pulled from your GitHub repository.
2. **Dependencies**: The necessary dependencies will be installed.
3. **Tests**: Tests will be executed using the configured test runner.
4. **SonarQube Analysis**: The code will be analyzed by SonarQube, and any issues detected will be available in the SonarQube dashboard.

You can check the results of the analysis in your SonarQube dashboard, which will display code quality metrics and suggest improvements.

---

## **Troubleshooting**

- **SonarQube Configuration**: Ensure that the SonarQube server is properly configured in Jenkins with the correct server name and credentials.
- **NodeJS Version**: Ensure that the `nodejs` plugin is properly installed in Jenkins and that the correct Node version (`nodejs20` in this case) is available.
- **Test Failures**: If the tests fail, check the test output in the Jenkins console to diagnose the issues.

---

