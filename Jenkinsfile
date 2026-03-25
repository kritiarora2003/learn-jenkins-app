pipeline {
    agent any

    environment {
        APP_NAME = 'myjenkinsapp'
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "ap-south-1"
        AWS_ECS_CLUSTER = 'learnJenkinsApp-cluster-prod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'learnJenkinsApp-TaskDefinition-Prod'
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

        stage("Build Docker Image") {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    docker build -t $APP_NAME:$REACT_APP_VERSION .
                '''
            }
        }

        stage("deploy to aws") {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services
                    '''
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