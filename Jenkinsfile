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

                git branch: 'develop',
                    url: "${GIT_REPO}"
                
                sh '''
                    rm -rf config
                
                    git clone -b staging https://github.com/AsuruDev/todo-list-aws-config.git config
                
                    cp config/samconfig.toml .
                '''

            }
        }

        stage('Static Test') {

            steps {
                sh '''
                    mkdir -p reports
        
                    echo "Running Flake8..."
                    python3 -m flake8 src --format=pylint > reports/flake8.out || true
        
                    echo "Running Bandit..."
                    python3 -m bandit -r src \
                        -f custom \
                        --msg-template "{abspath}:{line}: [{test_id}] {msg}" \
                        -o reports/bandit.out || true
                '''
            }
        
            post {
                always {
        
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            flake8(name: 'Flake8', pattern: 'reports/flake8.out'),
                            pyLint(name: 'Bandit', pattern: 'reports/bandit.out')
                        ]
                    )
                }
            }
        }

        stage('Deploy') {

            steps {
        
                sh '''
                    sam build
                    sam validate
        
                    sam deploy \
                      --config-env staging \
                      --resolve-s3 \
                      --no-confirm-changeset \
                      --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Get API URL') {

            steps {

                sh '''
                    aws cloudformation describe-stacks \
                    --stack-name todo-list-aws-staging \
                    --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                    --output text > api_url.txt

                    echo "API URL:"
                    cat api_url.txt
                '''
            }
        }

        stage('Rest Test') {

            steps {

                sh '''
                    API_URL=$(cat api_url.txt)

                    echo "Testing endpoint:"
                    echo $API_URL

                    curl -f \
                        "$API_URL/todos"
                '''
            }
        }

        stage('Promote') {

        steps {
    
            withCredentials([
    
                usernamePassword(
    
                    credentialsId: 'github-creds',
    
                    usernameVariable: 'GIT_USER',
    
                    passwordVariable: 'GIT_PASS'
    
                )
    
            ]) {
    
                sh '''
    
                    set -e
    
                    git config user.name "jenkins"
    
                    git config user.email "jenkins@local"
    
                    git fetch origin
    
                    git checkout master
    
                    git pull origin master
    
                    git merge origin/develop --no-commit --no-ff
    
                    # Mantener SIEMPRE el Jenkinsfile de master
    
                    git checkout --ours Jenkinsfile
    
                    git add Jenkinsfile
    
                    git commit -m "Promote develop to master"
    
                    git push \
    
                      https://${GIT_USER}:${GIT_PASS}@github.com/AsuruDev/todo-list-aws.git \
    
                      master
    
                '''
                }
            }
        }
    }
}
