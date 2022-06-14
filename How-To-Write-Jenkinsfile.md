# 1
```
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
            }
        }
        
        stage('Build') {
            steps{
                // Do your build task
            }
        }
        
        stage('Test') {
            steps{
                // Run unit tests, integration tests, and/or e2e tests
            }
        }
        
        stage('Publish') {
            steps{
                // Publish your artifacts to somewhere.
                // However, in our hands-on, you just need to print the artifact list by Linux command 'ls -la'
            }
        }

        post {
            always { 
                echo 'I will always say Hello again!'
            }
        }
    }
}

```

# 2

```
pipeline {
 agent any
 
 environment {
     CI = 'true'
 }
 stages {
     stage('Build') {
         steps {
             sh 'npm install'
         }
     }
     stage('Test') {
         steps {
             sh './jenkins/scripts/test.sh'
         }
     }
     stage('Deliver for development') {
         when {
             branch 'development'
         }
         steps {
             sh './jenkins/scripts/deliver-for-development.sh'
             input message: 'Finished using the web site? (Click "Proceed" to continue)'
             sh './jenkins/scripts/kill.sh'
         }
     }
     stage('Deploy for production') {
         when {
             branch 'production'
         }
         steps {
             sh './jenkins/scripts/deploy-for-production.sh'
             input message: 'Finished using the web site? (Click "Proceed" to continue)'
             sh './jenkins/scripts/kill.sh'
         }
     }
 }

```

# 3
```
pipeline {
    agent any
    
    stages {
        stage('Build Docker image') {
            steps {
                sh 'docker build -t docker-getting-started .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -dp 3000:3000 docker-getting-started'
            }
          }
    }
}

```













# BuildKit for Docker.md

```
https://www.youtube.com/watch?v=9CRDMs5D3kM
https://github.com/darinpope/jenkins-example-buildkit
```
