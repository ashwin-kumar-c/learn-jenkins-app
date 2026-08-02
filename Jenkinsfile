pipeline {
    agent any

    environment {
        REACT_APP_VERSION= "1.0.$BUILD_ID"
    }

    stages {

        stage('Update to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            environment {
                AWS_DEFAULT_REGION = 'us-east-1'
                AWS_S3_BUCKET = 's3-for-jenkins-dock'
            }


            steps {
                withCredentials([
                usernamePassword(
                credentialsId: 'aws-creds',
                usernameVariable: 'AWS_ACCESS_KEY_ID',
                passwordVariable: 'AWS_SECRET_ACCESS_KEY'
            )
        ]) {
            sh '''
                aws --version
                aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                aws ecs update-service \
                --cluster jenkins-cluster \
                --service Jenkins-Service-Prod \
                --task-definition Jenkins-TaskDefinition-Prod:2
            '''
        }
            }
        }

        // stage('Build') {

        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             echo 'Building Started ....'
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }

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
        //                     echo 'Testing...'
        //                     #test -f build/index.html
        //                     npm test
        //                 '''
        //             }

        //                 post {
        //                     always {
        //                         junit 'jest-results/junit.xml'
        //                     }
        //                 }
        //         }

        //         stage('E2E') {

        //             agent {
        //                 docker {
        //                     image 'demo-playwright'
        //                     reuseNode true
        //                 }
        //             }
                    
        //             steps {
        //                 sh '''
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=line
        //                 '''
        //             }

        //                 post {
        //                     always {
        //                         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
        //                     }
        //                 }
        //         }
        //     }
        // }

        // stage('Deploy to Staging') {

        //     agent {
        //         docker {
        //             image 'demo-playwright'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             netlify --version
        //             echo "Deploying to Production. Project ID: $NETLIFY_PROJECT_ID"
        //             netlify deploy \
        //                 --no-build \
        //                 --dir=build \
        //                 --site="$NETLIFY_PROJECT_ID" \
        //                 --auth="$NETLIFY_AUTH_TOKEN"
        //         '''
        //     }
        // }

        // stage('Deploy to AWS Prod') {
        //     agent {
        //         docker {
        //             image 'amazon/aws-cli'
        //             reuseNode true
        //             args "--entrypoint=''"
        //         }
        //     }

        //     environment {
        //         AWS_DEFAULT_REGION = 'ap-south-1'
        //         AWS_S3_BUCKET = 's3-for-jenkins-dock'
        //     }


        //     steps {
        //         withCredentials([
        //         usernamePassword(
        //         credentialsId: 'aws-creds',
        //         usernameVariable: 'AWS_ACCESS_KEY_ID',
        //         passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        //     )
        // ]) {
        //     sh '''
        //         aws --version
        //         aws s3 sync build s3://$AWS_S3_BUCKET --delete
        //     '''
        // }
        //     }
        // }

        // stage('E2E Test') {

        //     agent {
        //         docker {
        //             image 'demo-playwright'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             npx playwright test --reporter=line
        //         '''
        //     }

        //         post {
        //             always {
        //                 publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod', reportTitles: '', useWrapperFileDirectly: true])
        //             }
        //         }
        // }

    }

}
