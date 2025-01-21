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

        stage ('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                npm install netlify-cli
                npm install node-jq
                node_modules/.bin/netlify --version
                echo "deployin to production site id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                echo "small change"
                '''
                script {
                env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",returnStdout: true)
            }
            }
        }
        
        stage ('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                     CI_ENVIRONMENT_URL = "$(env.STAGING_URL)"
            }
            steps{
                echo 'Test E2E'
                sh '''
                npx playwright test --reporter=html
                '''
            }
            post {
            always {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
            }

         stage('Approval') {
            steps {
                timeout(1) {
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure to deploy'
                }
                
            }
        }

        stage ('Deploy Prod') {
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

        stage ('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                     CI_ENVIRONMENT_URL = 'https://cheery-zuccutto-854c34.netlify.app'
            }
            steps{
                echo 'Test E2E'
                sh '''
                npx playwright test --reporter=html
                '''
            }
            post {
            always {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
            }
    }
}