pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '50f59c3f-5b4d-40fb-9f77-e87d867248ef'
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
        stage('Deploy') {
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