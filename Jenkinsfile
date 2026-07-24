pipeline {
    agent any

    environment {
        NETLIFY_PROJECT_ID= '7d4fbe0f-b112-4f1d-a79d-85547865125c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=line
                        '''
                    }

                        post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                }
            }
        }

        stage('Deploy') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to Production. Project ID: $NETLIFY_PROJECT_ID"
                    node_modules/.bin/netlify deploy \
                        --prod \
                        --dir=build \
                        --site="$NETLIFY_PROJECT_ID" \
                        --auth="$NETLIFY_AUTH_TOKEN"
                '''
            }
        }

    }

}
