pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                sh "echo $WORKSPACE"
                cleanWs()
                // Clonamos repositorio
                sh "git clone --branch master https://github.com/unir-mgonz/todo-list-aws.git"
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
                                pwd

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
    }
}