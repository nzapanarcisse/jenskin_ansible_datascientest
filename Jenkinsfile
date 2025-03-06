/* import shared library */
@Library('narcisse-shared-library') _ 
pipeline {
    agent none
    
    stages {
        stage('Parallel Tasks') {
            parallel {
                
                stage('Check yaml syntax') {
                    agent { docker { image 'sdesbure/yamllint' } }
                    steps {
                        sh 'yamllint --version'
                        sh 'yamllint \${WORKSPACE}'
                    }
                    }
                stage('Check markdown syntax') {
                    agent { docker { image 'ruby:alpine' } }
                    steps {
                        sh 'apk --no-cache add git'
                        sh 'gem install mdl'
                        sh 'mdl --version'
                        sh 'mdl --style all --warnings --git-recurse \${WORKSPACE}'
                    }
                }
            }
        }
        
        stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
            }
        }
        stage('Test and deploy the application') {
            environment {
                SUDOPASS = credentials('sudopass')
            }
            agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
            stages {

               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh '''
                       apt-get update
                       apt-get install -y sshpass
                       ansible-playbook  -i hosts.yml --vault-password-file vault.key  --extra-vars "ansible_sudo_pass=$SUDOPASS" deploy.yml
                       '''
                   }
               }
            }
          }
      }
       post {
        always{
           script{
               slackNotifier currentBuild.result}
                   }
           }

    
    /*  post {

           failure {
                 slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
           }
           success {
                         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) - PROD URL => http://185 , STAGING URL => http://25")
           }
         }*/

    }
