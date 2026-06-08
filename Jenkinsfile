pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                sh "echo $WORKSPACE"
                cleanWs()
                // Clonamos repositorio
                sh "git clone --branch develop git@github.com:unir-mgonz/todo-list-aws.git"
                sh "wget -O todo-list-aws/samconfig.toml https://raw.githubusercontent.com/unir-mgonz/todo-list-aws-config/refs/heads/staging/samconfig.toml"
            }
        }
        stage('Static tests') {
            parallel {
                stage('Flake8') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh'''
                                export PYTHONPATH="$WORKSPACE/todo-list-aws"
                                cd $PYTHONPATH

                                touch ../results-flake8.txt
                                /usr/local/bin/python/bin/python -m flake8 --exit-zero --format=pylint src/ | tee ../results-flake8.txt
                            '''
                        }
                        recordIssues tools: [flake8(name:'Flake8', pattern: 'results-flake8.txt')]
                    }
                }
                stage('Bandit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh'''
                                export PYTHONPATH="$WORKSPACE/todo-list-aws"
                                cd $PYTHONPATH

                                touch ../results-bandit.json
                                /usr/local/bin/python/bin/python -m bandit -r src/ -f json  -o ../results-bandit.json
                            '''
                        }
                        recordIssues tools: [pyLint(name:'Bandit', pattern: 'results-bandit.json')]
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh'''
                    cd $WORKSPACE/todo-list-aws
                    sam build
                    sam validate --region us-east-1
                    sam deploy --no-fail-on-empty-changeset --config-env staging
                    echo $?

                    # aws cloudformation describe-stacks help
                    # Usando output json luego se pueden hacer queries con JMESPath
                    aws cloudformation describe-stacks --stack-name todo-list-aws-staging --output text | grep BaseUrlApi | awk '{print $NF}' | tee ../BaseUrlApi.txt
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
                    /usr/local/bin/python/bin/python -m pytest --junitxml=results-rest.xml todo-list-aws/test/integration/todoApiTest.py
                '''
                junit 'results-rest.xml'
            }
        }
        stage('Promote') {
            steps {
                sh'''
                    cd $WORKSPACE/todo-list-aws
                    set -e
                    
                    git fetch origin 
                    git checkout master
                    git pull origin master
                    
                    git merge origin/develop --no-commit --no-ff || true

                    git checkout origin/master -- Jenkinsfile Jenkinsfile_AGENTES # Escogemos la version de Jenkinsfile ya existente en master
                    git add Jenkinsfile Jenkinsfile_AGENTES

                    git commit -m "Promote development branch to main"


                    git push git@github.com:unir-mgonz/todo-list-aws.git master
                '''
            }
        }
    }
}