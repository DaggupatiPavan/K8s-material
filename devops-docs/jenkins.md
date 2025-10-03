# Jenkins - Continuous Integration/Continuous Deployment

## Overview

Jenkins is an open-source automation server that enables developers to build, test, and deploy their software. It provides hundreds of plugins to support building, deploying, and automating any project.

### Key Features
- **Open Source**: Free and community-driven
- **Extensible**: 1800+ plugins available
- **Distributed**: Master-agent architecture
- **Pipeline as Code**: Jenkinsfile for pipeline definition
- **Easy Integration**: Works with various DevOps tools

## Core Concepts

### Jenkins Architecture
- **Jenkins Master**: Central server that coordinates builds
- **Jenkins Agents/Slaves**: Execute build tasks
- **Plugins**: Extend functionality
- **Jobs/Projects**: Build automation tasks
- **Pipeline**: Automated process for CD/CI

### Job Types
1. **Freestyle Projects**: Simple, UI-configured jobs
2. **Pipeline Projects**: Code-defined pipelines
3. **Multi-configuration Projects**: Test across multiple configurations
4. **Folder**: Organize related jobs

### Pipeline Concepts
- **Stage**: Logical division of pipeline (e.g., Build, Test, Deploy)
- **Step**: Single task within a stage
- **Node**: Machine where pipeline executes
- **Agent**: Specifies where pipeline runs

## Installation & Setup

### System Requirements
- **Java**: Jenkins requires Java 8 or later
- **Memory**: Minimum 2GB RAM (4GB recommended)
- **Disk**: Minimum 10GB free space
- **OS**: Windows, macOS, Linux

### Installation Methods

#### Using Package Manager (Ubuntu/Debian)
```bash
# Install Java
sudo apt update
sudo apt install openjdk-11-jdk

# Add Jenkins repository
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Install Jenkins
sudo apt update
sudo apt install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

#### Using Docker
```bash
# Run Jenkins in Docker
docker run -d -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts

# With persistent volume
docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

#### Using WAR File
```bash
# Download WAR file
wget http://updates.jenkins-ci.org/download/war/2.319.1/jenkins.war

# Run Jenkins
java -jar jenkins.war --httpPort=8080
```

## Jenkins Pipeline

### Declarative Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        PATH = "/usr/local/bin:${env.PATH}"
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying to production...'
                sh './deploy.sh'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

### Scripted Pipeline
```groovy
node {
    stage('Build') {
        echo 'Building application...'
        sh 'mvn clean package'
    }
    
    stage('Test') {
        echo 'Running tests...'
        sh 'mvn test'
    }
    
    stage('Deploy') {
        echo 'Deploying to production...'
        sh './deploy.sh'
    }
}
```

## Key Plugins

### Essential Plugins
- **Pipeline**: Defines build pipelines as code
- **Git**: Integrates with Git repositories
- **GitHub**: GitHub integration
- **Docker**: Docker build and run support
- **Kubernetes**: Kubernetes integration
- **Maven**: Maven project support
- **NodeJS**: Node.js build support
- **Email Extension**: Email notifications
- **Slack**: Slack notifications
- **SonarQube**: Code quality analysis

### Plugin Management
```bash
# Install plugins via CLI (using Jenkins CLI)
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin git docker

# List installed plugins
java -jar jenkins-cli.jar -s http://localhost:8080 list-plugins
```

## Jenkins Configuration

### System Configuration
- **System Settings**: Configure Jenkins URL, admin email
- **Global Tool Configuration**: Configure JDK, Maven, Git, etc.
- **Plugin Management**: Install and manage plugins
- **Security Configuration**: Set up authentication and authorization

### Security Configuration
```groovy
// Jenkins security configuration example
import jenkins.model.*
import hudson.security.*

def instance = Jenkins.getInstance()

// Enable security
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "password")
instance.setSecurityRealm(hudsonRealm)

// Set authorization strategy
def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
instance.setAuthorizationStrategy(strategy)

instance.save()
```

## Jenkinsfile Best Practices

### Structure
```groovy
pipeline {
    agent {
        label 'linux'
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
        retry(3)
        disableConcurrentBuilds()
    }
    
    environment {
        APP_NAME = 'my-app'
        VERSION = '${BUILD_NUMBER}'
    }
    
    triggers {
        cron('H 2 * * *')  // Run at 2 AM daily
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn integration-test'
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            emailext (
                subject: "Build Success: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build successful. View: ${env.BUILD_URL}",
                to: "team@example.com"
            )
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. View: ${env.BUILD_URL}",
                to: "team@example.com"
            )
        }
    }
}
```

## Jenkins Agent Configuration

### SSH Agent Setup
```groovy
// Configure SSH agent in Jenkins
node {
    // Define agent
    def agentName = 'build-agent'
    
    stage('Configure Agent') {
        steps {
            echo 'Configuring SSH agent...'
            // SSH agent configuration done in Jenkins UI
        }
    }
}
```

### Docker Agent
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8.4-openjdk-11'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

## Integration with DevOps Tools

### Git Integration
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/user/repo.git', 
                    branch: 'main',
                    credentialsId: 'github-creds'
            }
        }
    }
}
```

### Docker Integration
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('my-app:${BUILD_NUMBER}')
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.example.com', 'docker-creds') {
                        docker.image('my-app:${BUILD_NUMBER}').push()
                    }
                }
            }
        }
    }
}
```

### Kubernetes Integration
```groovy
pipeline {
    agent any
    
    stages {
        stage('Deploy to Kubernetes') {
            steps {
                kubernetesDeploy(
                    configs: 'k8s/*.yaml',
                    kubeconfigId: 'k8s-config'
                )
            }
        }
    }
}
```

## Monitoring & Logging

### Build Monitoring
- **Build History**: View past builds
- **Console Output**: Check build logs
- **Build Trends**: Analyze build success/failure rates
- **Metrics**: Monitor build times and resource usage

### Logging Configuration
```groovy
pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Log Example') {
            steps {
                echo 'INFO: Starting build process'
                echo 'DEBUG: Current directory contents'
                sh 'ls -la'
            }
        }
    }
}
```

## Interview Questions

### Beginner Level
1. **What is Jenkins?**
   - Jenkins is an open-source automation server used for continuous integration and continuous deployment

2. **What are the main features of Jenkins?**
   - Continuous integration, continuous delivery, extensive plugin ecosystem, distributed builds

3. **What is a Jenkins pipeline?**
   - A sequence of jobs or events that are interconnected and executed in a specific order

4. **What is the difference between freestyle project and pipeline in Jenkins?**
   - Freestyle projects are configured through UI, while pipelines are defined as code

### Intermediate Level
1. **What is Jenkinsfile?**
   - A text file that contains the definition of a Jenkins pipeline

2. **What are the different types of Jenkins pipelines?**
   - Declarative pipeline and scripted pipeline

3. **What is the role of Jenkins master and agent?**
   - Master coordinates builds and serves the web UI, agents execute build tasks

4. **How do you secure Jenkins?**
   - Configure authentication, authorization, CSRF protection, and SSL

### Advanced Level
1. **How do you handle environment-specific configurations in Jenkins?**
   - Use environment variables, configuration files, or parameterized builds

2. **What is Jenkins shared library?**
   - A collection of reusable pipeline code that can be shared across multiple pipelines

3. **How do you implement blue-green deployment in Jenkins?**
   - Use pipeline stages to deploy to two identical environments and switch traffic

4. **What is the difference between Jenkins and other CI/CD tools?**
   - Jenkins is highly customizable with plugins, while others may have more specific use cases

## Best Practices

### Pipeline Design
- **Modular Design**: Break complex pipelines into smaller stages
- **Error Handling**: Implement proper error handling and notifications
- **Idempotency**: Ensure pipelines can be run multiple times safely
- **Version Control**: Store Jenkinsfile in version control

### Security
- **Least Privilege**: Use minimal required permissions
- **Credential Management**: Use Jenkins credential store
- **Regular Updates**: Keep Jenkins and plugins updated
- **Audit Logs**: Enable and monitor audit logs

### Performance
- **Agent Distribution**: Use multiple agents for parallel builds
- **Resource Management**: Monitor and manage agent resources
- **Pipeline Optimization**: Optimize build steps and dependencies
- **Cleanup**: Regular cleanup of old builds and artifacts

## Troubleshooting

### Common Issues
```bash
# Check Jenkins status
sudo systemctl status jenkins

# Check Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log

# Restart Jenkins
sudo systemctl restart jenkins

# Check Java version
java -version

# Check port availability
netstat -tlnp | grep 8080
```

### Pipeline Debugging
```groovy
pipeline {
    agent any
    
    stages {
        stage('Debug') {
            steps {
                script {
                    // Print environment variables
                    sh 'env'
                    
                    // Print current directory
                    sh 'pwd'
                    
                    // Print Jenkins version
                    echo "Jenkins version: ${jenkins.model.Jenkins.instance.version}"
                }
            }
        }
    }
}
```

## Resources

### Official Documentation
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)

### Learning Resources
- [Jenkins Official Tutorial](https://www.jenkins.io/doc/book/)
- [Jenkins Pipeline Examples](https://github.com/jenkinsci/pipeline-examples)

### Community
- [Jenkins Community](https://community.jenkins.io/)
- [Jenkins Stack Overflow](https://stackoverflow.com/questions/tagged/jenkins)