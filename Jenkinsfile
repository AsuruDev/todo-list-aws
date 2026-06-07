pipeline {

    agent any

    environment {
        GIT_REPO   = 'https://github.com/AsuruDev/todo-list-aws.git'
        STACK_NAME = 'todo-list-aws'
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master',
                    url: "${GIT_REPO}"

                sh '''
                    rm -rf config

                    git clone -b production https://github.com/AsuruDev/todo-list-aws-config.git config

                    cp config/samconfig.toml .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build

                    sam deploy \
                        --config-env production \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --resolve-s3 \
                        --parameter-overrides Stage=production
                '''
            }
        }

        stage('Get API URL') {
            steps {
                sh '''
                    aws cloudformation describe-stacks \
                        --stack-name todo-list-aws-production \
                        --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                        --output text > api_url.txt
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    API_URL=$(cat api_url.txt)

                    echo "GET only test"

                    curl -f $API_URL/todos
                '''
            }
        }
    }
}
