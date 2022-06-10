# -freestyle-Jenkins-Azure-NodeJS-NoAgent-NoDocker/

# Work flow
* Azure VM to host Jenkins
  * create
  * DNS
  * port open
  * allow speficic IP address to acess VM (my own IP address)
  
* VM configuration
  * ssh to VM
  * install java beacause of jenkins
  * install jenkins
  * home directory need to relocate? 
  
* Manage Jenkins
  * user/group/role --permission assign
  * plugin -blue ocean
  * reverse proxy broken??
  * 

  
* connect to GitHub Repo blue Ocean for multi-branches
  * use webhook (No) or token (yes) ?
  * skip indexing?
  * where is the Jenkinsfile
  * !!!SET Scan Repo Trigger PERIODCALLY to see whether branch has changed in settings of Blue Ocean!!!



* the application ( On jenkins locally and no docker atm)
  * nodeJS -  npm ? package install ?
  * how to run nodejs test
  * how to run nodejs
  *  s


*  Jenkinsfile
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
}
   
   ```
