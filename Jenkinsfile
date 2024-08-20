pipeline {
    agent any

    environment {
        // NETLIFY_SITE_ID = '2b53ebcb-9158-44c9-a0c8-de2d48215276'
        // NETLIFY_AUTH_TOKEN = credentials('netlify_tok')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''" 
                }
            }
            // environment {
            //     // AWS_S3_BUCKET = 'ci-cd-bucket-20240818'
            // }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster JenkinsApp-Cluster-Prod --service JenkinsApp-Service-Prod --task-definition JenkinsApp-Cluster-TaskDefinition-Prod:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster JenkinsApp-Cluster-Prod --services JenkinsApp-Service-Prod
                    '''
                }              
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'playwright-image'
                    reuseNode true
                }
            }
            steps {
            sh '''
                node --version
                npm --version
                npm ci
                npm run build
                ls -la 
            '''
            }
        }
        
        // stage('Run Tests') {
        //     parallel {
        //         stage('Unit test') {
        //             agent {
        //                 docker {
        //                     image 'playwright-image'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     echo "Test stage"
        //                     test -f build/index.html
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
        //                     image 'playwright-image'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     serve -s build & 
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
        //                 }
        //             }
        //         }
        //     }    
        // }   

    /*    stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json             
                '''
                script {
                    env.STAGING_URL = sh (script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)  
                } 
            }
            
        }
       */ 

    //     stage('Deploy staging') {
    //         agent {
    //             docker {
    //                 image 'playwright-image'
    //                 reuseNode true
    //             }
    //         }

    //         environment {
    //            CI_ENVIRONMENT_URL = "TO_BE_SPECIFIED"
    //         }
            
    //         steps {
    //             sh '''                 
    //                 netlify --version
    //                 echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
    //                 netlify status
    //                 netlify deploy --dir=build --json > deploy-output.json 
    //                 CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
    //                 npx playwright test --reporter=html
    //             '''
    //         }
    //         post {
    //             always {
    //                 publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
    //             }
    //         }
    //     }
    // /*
    //     stage('Approval') {
    //         steps {
    //             timeout(time: 1, unit: 'MINUTES') {
    //             input message: 'Are you ready to deploy?', ok: 'Yes, why not?'
    //             }
               
    //         }
    //     }   
    // */                      
                
    //     stage('Deploy prod') {
    //         agent {
    //             docker {
    //                 image 'playwright-image'
    //                 reuseNode true
    //             }
    //         }

    //         environment {
    //            CI_ENVIRONMENT_URL = 'https://stalwart-bombolone-afc733.netlify.app'
    //         }
            
    //         steps {
    //             sh '''
    //                 npm --version
    //                 netlify --version
    //                 echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
    //                 netlify status
    //                 netlify deploy --dir=build --prod
    //                 npx playwright test --reporter=html
    //             '''
    //         }
    //         post {
    //             always {
    //                 publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
    //             }
    //         }
    //     }
    }
}
