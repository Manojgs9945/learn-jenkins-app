pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                echo 'Hello World'
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
        stage('E2E'){
            agent {
                docker{
                    image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                    reuseNode true
                }
            }
            steps{
                sh '''
                npm install serve
                npx playwright install
                node_modules/.bin/serve -s build &
                sleep 20
                npx playwright test --reporter=html
                '''
            }
        }
        stage('Publish HTML Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'playwright-report',  // Directory containing your HTML report
                    reportFiles: 'index.html',       // Entry point file of your report
                    reportName: 'Playwright HTML Report' // Report name displayed in Jenkins
                ])
            }
        }
    }
}