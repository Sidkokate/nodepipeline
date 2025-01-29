# nodepipeline
create folder on your local machine nodepipeline
cd nodepipeline
git init
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Sidkokate/nodepipeline.git
git push -u origin main
# orr simply do git clone https://github.com/Sidkokate/nodepipeline.git
on jenkins server
 sudo systemctl start jenkins
 in jenkins 
 Jenkins --> new project (nodepipeline type pipeline)
pipeline syntax --> checkout: check out from version control  --> git : git repo genrate pip script

script 
---------------------------------------------------
pipeline {
    agent any
    tools{
        nodejs 'mynode'
    }
    stages {
        stage('git cloning') {
            steps {
                echo 'cloning files from github'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Sidkokate/nodepipeline.git']])
            }
        }
        stage('Build') {
            steps {
                echo 'Building nodejs project'
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing project'
                sh './node_modules/mocha/bin/_mocha --exit ./test/test.js'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying nodejs project on live server'
                script{
                    sshagent(['e68049ab-298e-4eb8-aa8c-298162d28146']) {  
                     sh '''
                           ssh -o StrictHostKeyChecking=no ubuntu@43.204.114.125<<EOF
                            cd /home/ubuntu/nodepipeline/
                            git pull https://github.com/Shridharkokate/nodepipeline.git
                            npm install
                            sudo npm install -g pm2
                            pm2 restart index.js || pm2 start index.js
		                    exit
                            EOF     
                           '''
                   }
                }
            }
        }
    }
}
---------------------------------------------------
# deploy stage 
add GitHub hook : on GitHub
add agent any if no agent is spacify
add required tool in pipeline 
add ssh agent : Jenkins--> manage_jenkins --> plugins --> ssh agent
add ssh server : Jenkins--> manage_jenkins --> system --> ssh server creation 
get sshagent id : Jenkins--> manage_jenkins --> security --> credential
