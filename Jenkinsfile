pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '50f59c3f-5b4d-40fb-9f77-e87d867248ef'
        NETLIFY_AUTH_TOKEN  = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage("Docker") {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

        stage("build") {
            agent  {
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
                    ls -ls
                '''
            }
        }
        
        stage('Tests') {
            parallel {
                stage("test") {
                    agent  {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh 'test -f build/index.html'
                        sh 'npm test'
                    }
                }

                stage("e2e") {
                    agent  {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }

        stage("deploy staging e2e") {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npx netlify deploy --dir=build --no-build --json > deploy_output.json
                '''

                script {
                    env.CI_ENVIRONMENT_URL = sh(
                        script: "node_modules/.bin/node-jq -r '.deploy_url' deploy_output.json",
                        returnStdout: true
                    ).trim()
                }

                sh '''
                    echo "Testing staging URL: $CI_ENVIRONMENT_URL"
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input(
        //                 message: 'Do you wish to deploy to production?',
        //                 ok: 'Yes, I am sure!'
        //             )
        //         }
        //     }
        // }

        stage("deploy prod e2e") {
            agent  {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://animated-monstera-5a9e93.netlify.app'
            }

            steps {
                sh '''
                    echo "deploying to prod##############"
                    npx netlify deploy --dir=build --prod --no-build
                    serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}