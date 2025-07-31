pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '36c91d6c-cd7e-413e-8f3a-ef5aaf3d06d2'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.2.3'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
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

        stage('Run Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            # test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                icon: '',
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Local',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }


        stage('Deploy staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

             environment {
                CI_ENVIRONMENT_URL =  'STAGING_URL_TO_BE_SET'
             }


            steps {
                sh '''
                    npm install netlify-cli
                    npm install node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging : Site ID  : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=line
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Stating E2E Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }        

        /*stage('Approval to Production'){
            steps {
                    timeout(time: 1, unit: 'HOURS') {
                    input message: 'Ready to deploy production?', ok: 'Yes, I am sure I want to deploy'
                    }
                }
        }*/


        



        stage('Deploy prod') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL = 'YOUR NETLIFY URL'
                    }

                    steps {
                        sh '''
                            node --version
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod --no-build
                            npx playwright test  --reporter=html
                        '''
                    }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Prod E2E Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }


    }
}
