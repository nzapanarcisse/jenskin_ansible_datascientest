/* import shared library */
/*@Library('narcisse-shared-library') _*/
pipeline {
    agent none
    stages {
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
      /*
      post {
        always{
           script{
               slackNotifier currentBuild.result}
                   }
           }
     */
     post{
        SUCCESS {
            slackSend color: "good", message: "Narcisse => CONGRATULATION: Job ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was successful ! more info ${env.BUILD_URL}"
          }
        FAILURE {
            slackSend color: "danger", message: "Narcisse => BAD NEWS:Job ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was failed ! more info ${env.BUILD_URL}"
          }
        UNSTABLE {
            slackSend color: "warning", message: "Narcisse => BAD NEWS:Job ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was unstable ! more info ${env.BUILD_URL}"
          }
        DANGER {
            slackSend color: "danger", message: "Narcisse => BAD NEWS:Job ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} its result was unclear ! more info ${env.BUILD_URL}"
          }
     }
    }