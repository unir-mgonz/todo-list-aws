pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                sh "echo $WORKSPACE"
                cleanWs()
                // Clonamos repositorio
                sh "git clone --branch master git@github.com:unir-mgonz/todo-list-aws.git"
                sh "wget -O todo-list-aws/samconfig.toml https://raw.githubusercontent.com/unir-mgonz/todo-list-aws-config/refs/heads/production/samconfig.toml"
            }
        }
        stage('Deploy') {
            steps {
                sh'''
                    cd $WORKSPACE/todo-list-aws
                    sam build
                    sam validate --region us-east-1
                    sam deploy --no-fail-on-empty-changeset --config-env production
                    echo $?

                    # aws cloudformation describe-stacks help
                    # Usando output json luego se pueden hacer queries con JMESPath
                    aws cloudformation describe-stacks --stack-name todo-list-aws-production --output text | grep BaseUrlApi | awk '{print $NF}' | tee ../BaseUrlApi.txt
                '''
                stash includes: 'BaseUrlApi.txt', name: 'BaseUrlApi'
            }
        }

        stage('Rest test') {
            steps {
                unstash 'BaseUrlApi'
                sh'''
                    cd $WORKSPACE
                    export BASE_URL=$(cat BaseUrlApi.txt)
                    echo $BASE_URL
                    curl -f $BASE_URL/todos
                '''
            }
        }
    }
}