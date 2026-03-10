pipeline {
    agent any

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
                    npm install netlify-cli -g
                    netlify --version
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