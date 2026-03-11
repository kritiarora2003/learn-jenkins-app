pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '50f59c3f-5b4d-40fb-9f77-e87d867248ef'
        NETLIFY_AUTH_TOKEN  = credentials('netlify-token')
    }

    stages {
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install server
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install --save-dev netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "deploying to prostagingd##############"
                    node_modules/.bin/netlify status
                    npx netlify deploy --dir=build --no-build --json >> deploy_output.json
                    node_modules/.bin/node-jq -r. '.deploy_url' deploy_output.json
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input(
                        message: 'Do you wish to deploy to production?',
                        ok: 'Yes, I am sure!'
                    )
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install --save-dev netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to prod##############"
                    node_modules/.bin/netlify status
                    npx netlify deploy --dir=build --prod --no-build
                '''
            }
        }

        stage("prod e2e") {
            agent  {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://animated-monstera-5a9e93.netlify.app'
            }

            steps {
                sh '''
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
        }

    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}