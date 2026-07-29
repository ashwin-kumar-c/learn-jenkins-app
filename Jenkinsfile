pipeline {
    agent any

    environment {
        NETLIFY_PROJECT_ID= '7d4fbe0f-b112-4f1d-a79d-85547865125c'
        NETLIFY_AUTH_TOKEN= credentials('netlify-token')
        REACT_APP_VERSION= "1.0.$BUILD_ID"
    }

    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
            }
        }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', passwordVariable: 'WS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    aws s3 ls
                '''
            }
            }
        }

        stage('Build') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo 'Building Started ....'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
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
                            echo 'Testing...'
                            #test -f build/index.html
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
                            image 'demo-playwright'
                            reuseNode true
                        }
                    }
                    
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=line
                        '''
                    }

                        post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                }
            }
        }

        stage('Deploy to Staging') {

            agent {
                docker {
                    image 'demo-playwright'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Production. Project ID: $NETLIFY_PROJECT_ID"
                    netlify deploy \
                        --no-build \
                        --dir=build \
                        --site="$NETLIFY_PROJECT_ID" \
                        --auth="$NETLIFY_AUTH_TOKEN"
                '''
            }
        }

        stage('Prod Deploy & E2E Test') {

            agent {
                docker {
                    image 'demo-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL= 'https://gentle-quokka-1d7e01.netlify.app'
            }
            
            steps {
                sh '''
                node --version
                    netlify --version
                    echo "Deploying to Production. Project ID: $NETLIFY_PROJECT_ID"
                    netlify deploy \
                        --prod \
                        --no-build \
                        --dir=build \
                        --site="$NETLIFY_PROJECT_ID" \
                        --auth="$NETLIFY_AUTH_TOKEN"
                    npx playwright test --reporter=line
                '''
            }

                post {
                    always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
        }

    }

}
