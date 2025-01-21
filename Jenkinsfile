pipeline {
    agent any

    stages {
        stage ('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }

        }

        stage ('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                echo 'Test stage'
                sh '''
                test -f build/index.html
                npm test
                '''
            }

        }

        stage ('Test E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                echo 'Test E2E'
                sh '''
                npm install -g serve
                serve -s build
                npx plywright test
                '''
            }

        }
    }
     post {
            always {
                junit 'test-results/junit.xml'
            }
        }
}