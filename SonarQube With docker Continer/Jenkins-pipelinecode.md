
pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'Dev', url: 'https://github.com/Jagan-18/Multi-Tier-Python-Postgres-main.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                rm -rf # Remove any existing virtual environment
                python3 -m venv venv # Create a new virtual environment
                chmod -R 755 venv
                bash -c "
                source venv/bin/activate 
                pip install --upgrade pip
                pip install -r requirements.txt
                "
                '''
            }
        }
        stage('Tests') {
            steps {
                sh '''
                bash -c "
                source venv/bin/activate
                pytest --cov=app --cov-report=xml
                pytest --cov=app --cov-report=term-missing --disable-warnings
                "
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=python-project \
                    -Dsonar.projectKey=python-project \
                    -Dsonar.exclusions=venv/**  \
                    -Dsonar.sources=./ \
                     -Dsonar.branch.name=Dev \
                    -Dsonar.python.coverage.reportPaths=coverage.xml
                    '''
                }
            }
        }
    }
}




---
```markdown
# Multi-Tier Python with PostgreSQL

## Project Overview

This project is a Python application using a PostgreSQL database, designed with a multi-tier architecture. It is designed to be scalable, modular, and easy to maintain. The project is managed using Git and integrates with Jenkins for CI/CD pipelines.

## Features

- **Multi-Tier Architecture**: The application is built in separate layers to maintain clear separation of concerns.
- **Python and PostgreSQL**: The project utilizes Python for business logic and PostgreSQL for data storage.
- **Automated Testing**: Uses `pytest` for unit testing with coverage reporting.
- **SonarQube Integration**: Code quality analysis through SonarQube to monitor code health and maintainability.

## Prerequisites

Before setting up and running the project locally, ensure that you have the following installed:

- Python 3.x
- PostgreSQL
- Git
- Virtual Environment

## Setup Instructions

### 1. Clone the repository
```bash
git clone https://github.com/Jagan-18/Multi-Tier-Python-Postgres-main.git
cd Multi-Tier-Python-Postgres-main
```

### 2. Create and activate a virtual environment
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies
Make sure your virtual environment is activated, then install the required Python packages:
```bash
pip install -r requirements.txt
```

### 4. Set up PostgreSQL Database
Ensure you have PostgreSQL installed and running. You may need to configure a database for the application to connect to. 

1. Create a new PostgreSQL database:
   ```sql
   CREATE DATABASE my_database;
   ```

2. Update your connection settings in the project (in the relevant config or settings file).

### 5. Run the application
Once all dependencies are installed and the database is configured, you can start the application.

```bash
python app.py
```

## Running Tests

To run the tests, ensure your virtual environment is activated and then run:

```bash
pytest --cov=app --cov-report=xml
pytest --cov=app --cov-report=term-missing --disable-warnings
```

- The first command generates a coverage report in XML format.
- The second command runs the tests and shows a detailed output of missing lines of code coverage.

## SonarQube Integration

This project integrates with SonarQube for static code analysis. It checks code quality, coverage, and potential bugs or vulnerabilities.

### Running SonarQube Analysis

The Jenkins pipeline automates SonarQube analysis as part of the CI/CD process. If you wish to run it locally, you can use the following command (ensure `sonar-scanner` is installed):

```bash
$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=python-project \
                               -Dsonar.projectKey=python-project \
                               -Dsonar.exclusions=venv/** \
                               -Dsonar.sources=./ \
                               -Dsonar.branch.name=Dev \
                               -Dsonar.python.coverage.reportPaths=coverage.xml
```

## CI/CD Pipeline

This project is configured to use Jenkins for continuous integration and delivery. The pipeline automates the following processes:

1. **Git Checkout**: Pulls the latest code from the `Dev` branch.
2. **Install Dependencies**: Sets up a Python virtual environment and installs all dependencies.
3. **Tests**: Runs the tests and generates coverage reports.
4. **SonarQube Analysis**: Analyzes code quality and coverage through SonarQube.

### Jenkins Configuration

1. Install the necessary Jenkins plugins (e.g., SonarQube Scanner, Python, Git).
2. Configure the `sonar-scanner` tool in Jenkins under Global Tool Configuration.
3. Set up the `SonarQube` server in Jenkins under "Manage Jenkins > Configure System."
---