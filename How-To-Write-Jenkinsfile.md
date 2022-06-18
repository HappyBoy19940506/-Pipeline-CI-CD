```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```




# 1 https://github.com/HappyBoy19940506/ApiJenkinsSolution
```
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/Azure-Samples/openhack-devops-team.git'
            }
        }
        
        stage('npm install') {
            steps{
                dir("./apis/userprofile/") {
                    sh 'npm install'
                }
            }
        }
        
        stage('Tests') {
            steps{
                dir("./apis/userprofile/") {
                    sh 'npm test'
                }
            }
        }
        
        stage('npm coverage') {
            steps{
                dir("./apis/userprofile/") {
                    sh 'npm run cover'
                }
            }
        }

        stage('Publish') {
            steps{
                sh 'ls -la ./apis/userprofile/'
            }
        }
    }
}

```

# 2 https://github.com/RayMaAU/building-a-multibranch-pipeline-project/blob/master/Jenkinsfile

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
