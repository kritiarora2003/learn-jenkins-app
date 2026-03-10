pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '50f59c3f-5b4d-40fb-9f77-e87d867248ef'
        NETLIFY_AUTH_TOKEN  = credentials('netlify-token')
    }

    stages {

        stage("Build") {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage("unit-test") {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'test -f build/index.html'
                        sh 'CI=true npm test'
                    }
                }

                stage("e2e") {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install -g serve
                            serve -s build &
                            sleep 10
                            npx playwright test
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
                    npx netlify deploy --prod --dir=build
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