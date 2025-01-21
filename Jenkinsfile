pipeline {
    agent any

environment{
    NETLIFY_SITE_ID = '634932a0-8c4e-44c4-93f2-1cdc3b922c16'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
}

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
        stage ('Tests'){
            parallel{
            stage ('Unit Test') {
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
            post {
            always {
                junit 'jest-results/junit.xml'
            }
        }

        }

        stage ('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                echo 'Test E2E'
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10s
                npx playwright test --reporter=html
                '''
            }
            post {
            always {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
            }
        }
        }

        stage ('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "deployin to production site id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                echo "small change"
                '''
            }
        }
    }
}