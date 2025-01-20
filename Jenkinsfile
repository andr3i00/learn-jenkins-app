pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                echo 'Test npm'
                sh '''
                echo "without docker"
                ls -la
                touch container-no.txt
                '''
            }
        }
        
        stage('with docker') {
            agent {
                docker{
                image 'node:18-alpine'
                reuseNode true
                }
            }
            steps {
                echo 'Test npm'
                sh '''
                echo "with docker"
                npm --version
                ls -la
                touch container-yes.txt
                '''
            }
        }
    }
}
