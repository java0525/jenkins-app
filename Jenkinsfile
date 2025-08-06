pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '36c91d6c-cd7e-413e-8f3a-ef5aaf3d06d2'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-east-2'

        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod1'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {

        // stage('Docker'){
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }







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


        stage('Build Docker image') {
            // steps {
            //     sh 'docker build -t myjenkinsapp .'
            // }

             agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root -v  /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }

            steps{
                sh '''
                    apt update
                    amazon-linux-extras install docker -y
                    docker build -t myjenkinsapp .
                '''
            }
        }


        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            // environment {
            //     AWS_S3_BUCKET = 'learn-jenkins-20250803'
            // }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version

                    # aws s3 sync build s3://$AWS_S3_BUCKET

                    yum install jq -y

                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION 
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION 
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                '''
                }
                
            }
        }








        // stage('Tests') {
        //     parallel {
        //         stage('Unit Test') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     # test -f build/index.html
        //                     npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     # npm install serve
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     publishHTML([
        //                         allowMissing: false,
        //                         alwaysLinkToLastBuild: false,
        //                         icon: '',
        //                         keepAll: false,
        //                         reportDir: 'playwright-report',
        //                         reportFiles: 'index.html',
        //                         reportName: 'Playwright Local',
        //                         reportTitles: '',
        //                         useWrapperFileDirectly: true
        //                     ])
        //                 }
        //             }
        //         }
        //     }
        // }


        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

             environment {
                CI_ENVIRONMENT_URL =  'STAGING_URL_TO_BE_SET'
             }


            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging : Site ID  : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --no-build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
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
                        reportName: 'Staging E2E Report',
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



        // stage('Deploy prod'){
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }

        //            // environment {
        //                 // CI_ENVIRONMENT_URL = 'YOUR NETLIFY URL'
        //                 // CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
        //                 //sh 'CI_ENVIRONMENT_URL=$(node-jq -r \'.deploy_url\' deploy-output.json)'


        //             //}

        //             steps {

        //                 script {
        //                             env.CI_ENVIRONMENT_URL = sh(
        //                                 script: "node-jq -r '.deploy_url' deploy-output.json",
        //                                 returnStdout: true
        //                             ).trim()
        //                             echo "Production deployed at: ${env.CI_ENVIRONMENT_URL}"
        //                         }

        //                 sh '''
        //                     node --version
        //                     netlify --version
        //                     echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //                     netlify status
        //                     netlify deploy --dir=build --prod --no-build
        //                     npx playwright test  --reporter=html
        //                 '''
        //             }

        //     post {
        //         always {
        //             publishHTML([
        //                 allowMissing: false,
        //                 alwaysLinkToLastBuild: false,
        //                 icon: '',
        //                 keepAll: false,
        //                 reportDir: 'playwright-report',
        //                 reportFiles: 'index.html',
        //                 reportName: 'Prod E2E Report',
        //                 reportTitles: '',
        //                 useWrapperFileDirectly: true
        //             ])
        //         }
        //     }
        // }

    //     stage('Deploy prod') {
    //         agent {
    //             docker {
    //                 image 'my-playwright'
    //                 reuseNode true
    //             }
    // }

    // steps {
    //     sh '''
    //         node --version
    //         netlify --version
    //         echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
    //         netlify status
    //         netlify deploy --dir=build --prod --no-build --json > deploy-output.json
    //     '''

    //     script {
    //         env.CI_ENVIRONMENT_URL = sh(
    //             script: "jq -r '.deploy_url' deploy-output.json",
    //             returnStdout: true
    //         ).trim()
    //         echo "Production deployed at: ${env.CI_ENVIRONMENT_URL}"
    //     }

    //     sh '''
    //         npx playwright test --reporter=html
    //     '''
    // }

    // post {
    //     always {
    //         publishHTML([
    //             allowMissing: false,
    //             alwaysLinkToLastBuild: false,
    //             icon: '',
    //             keepAll: false,
    //             reportDir: 'playwright-report',
    //             reportFiles: 'index.html',
    //             reportName: 'Prod E2E Report',
    //             reportTitles: '',
    //             useWrapperFileDirectly: true
    //         ])
    //     }
    // }
}



    }

