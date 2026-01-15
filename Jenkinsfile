pipeline {
    agent any
    
    environment {
        REACT_APP_VERSION="1.2.$BUILD_ID"
        AWS_DEFAULT_REGION="us-east-1"
        
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

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-s3-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh ''' 
                        aws --version
                        yum install jq -y
                        LATEST_JENKINS_APP_TASK_DEF_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_JENKINS_APP_TASK_DEF_REVISION
                        aws ecs update-service --cluster Jenkins-App-Cluster-Prod --service learn-jenkins-app-prod-service --task-definition learn-jenkins-app-prod-taskDef:$LATEST_JENKINS_APP_TASK_DEF_REVISION
                    '''
                }
            }
        }

    }

}
