pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = 'fb6ed9c8-960a-46ec-bf5c-9abc5b166757'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
         }

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
                script {
                    env.MY_VAR1 = 'This is Variable1'
                    env.MY_VAR2 = sh(script: 'date', returnStdout: true)
                }
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
        // stage("all_test"){
        //     parallel{
        //         stage('unit_test'){
        //             steps{
        //                 sh'''
        //                 test -f public/index.html
        //                 '''
        //             }
        //         }
        //         stage('E2E'){
        //             agent {
        //                 docker{
        //                     image 'mcr.microsoft.com/playwright:v1.50.0-noble'
        //                     reuseNode true
        //                 }
        //             }
        //             steps{
        //                 sh '''
        //                 npm install serve
        //                 npx playwright install
        //                 node_modules/.bin/serve -s build &
        //                 sleep 20
        //                 npx playwright test --reporter=html
        //                 '''
        //             }
        //             post{
        //                 always{
        //                         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        //                     }
        //             }
        //          }
        //     }
        // }
            // stage('unit_test'){
            //     steps{
            //         sh'''
            //         test -f public/index.html 
            //         '''
            //     }
            // }
            // stage('E2E'){
            //     agent {
            //         docker{
            //             image 'mcr.microsoft.com/playwright:v1.50.0-noble'
            //             reuseNode true
            //         }
            //     }
            //     steps{
            //         sh '''
            //         npm install serve
            //         npx playwright install
            //         node_modules/.bin/serve -s build &
            //         sleep 20
            //         npx playwright test --reporter=html
            //         '''
            //     }
            // }

        stage('Deploy staging') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Deployment starts'
                sh '''
                   npm install netlify-cli node-jq
                   node_modules/.bin/netlify --version
                   echo "Site id deployed : $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy  --dir=build --json > deploy_status.json
                '''
            }
            script{
                env.ST_LINK = sh (script :"node_modules/.bin/node-jq -r '.deploy_url' deploy_status.json", returnStdout: true)
            }
        }
        stage('Staged E2E'){
                agent{
                    docker{
                            image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                            reuseNode true
                            }
                    }
                environment{
                        CI_ENVIRONMENT_URL = '${env.ST_LINK}'
                }
                steps{
                        sh '''
                        npx playwright install
                        npx playwright test --reporter=html
                        '''
                }
                post{
                        always{
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report E2E Stage', reportTitles: '', useWrapperFileDirectly: true])
                            }
                    }
                }
        stage('Approval'){
            steps{
                timeout(15){
                        input message: 'Are you ready for deploy?', ok: 'Proceed with deployment'
                        }
                }
        }
        stage('Deploy Prod') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Deployment starts'
                echo "MY_VAR1 is : ${env.MY_VAR1}"
                echo "Date from deploy staging : ${env.MY_VAR2}"
                sh '''
                   npm install netlify-cli 
                   node_modules/.bin/netlify --version
                   echo "Site id deployed : $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy  --dir=build --prod
                '''
            }
        }
        stage('Prod E2E'){
                agent{
                    docker{
                            image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                            reuseNode true
                            }
                    }
                environment{
                        CI_ENVIRONMENT_URL = 'https://firstmanualdepolyment.netlify.app/'
                }
                steps{
                        sh '''
                        npx playwright install
                        npx playwright test --reporter=html
                        '''
                }
                post{
                        always{
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report E2E Prod', reportTitles: '', useWrapperFileDirectly: true])
                            }
                    }
                }
        }
}
