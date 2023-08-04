    pipeline {
        agent {
            docker {
                image 'node:16-buster-slim' 
                args '-p 3000:3000' 
            }
        }
        stages {
            stage('Git Clone') { 
                steps {
                    git branch: 'react-app', url: 'https://github.com/Mirfani340/a428-cicd-labs'
                }
            }
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
        }
    }
