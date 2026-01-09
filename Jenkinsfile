pipeline {
    agent any
    
    environment {
        NETLIFY_SITE_ID = 'c178eec2-d93f-4f9f-a4f9-8ea4171b0f59'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION="1.2.$BUILD_ID"
    }

    stages {
        // Build stage using Node.js Docker image
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

        stage('Run Tests - parallel') {
            parallel{
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ''' 
                            echo "Running tests..."
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

                stage('Playwright E2E Tests') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ''' 
                            echo "Running E2E tests..."
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging and E2E Tests') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'TO_BE_SET'
            }

            steps {
                sh ''' 
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    echo "Running E2E Staging tests..."
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy Production and E2E Tests') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://enchanting-syrniki-8ec929-estio.netlify.app'
            }

            steps {
                sh ''' 
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod
                    netlify status

                    echo "Running E2E Production tests..."
                    npx playwright test --reporter=html
                '''
                // node_modules/.bin/netlify deploy --dir=build --prod
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Production', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }

}
