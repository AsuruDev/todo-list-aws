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
