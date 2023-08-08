    pipeline {
        agent {
            docker {
                image 'node:16-buster-slim' 
                args '-p 3000:3000' 
            }
        }
        stages {
            // stage('Git Clone') { 
            //     steps {
            //         git branch: 'react-app', url: '/home/dicoding/a428-cicd-labs'
            //     }
            // }
            stage('Build') { 
                steps {
                    // sh 'npm install -g yarn'
                    sh 'yarn install' 
                }
            }
            stage('Test') {
                steps {
                    sh './jenkins/scripts/test.sh'
                }
            }
            stage('Deploy') {
                steps {
                    sh './jenkins/scripts/deliver.sh'
                    input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)'
                    sh './jenkins/scripts/kill.sh'
                }
            }
        }
    }