pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                // Clonamos repositorio
                sh "git clone --branch master https://github.com/unir-mgonz/todo-list-aws.git"
            }
        }
        stage('Static tests') {
            paralell {
                stage('Flake8') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh'''
                                touch results-flake8.txt
                                python3 -m flake8 --exit-zero --format=pylint app > results-flake8.txt
                            '''
                        }
                        recordIssues tools: [flake8(Name:'Flake8', pattern: 'results-flake8.txt')]
                    }
                }
                stage('Bandit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh'''
                                touch results-bandit.xml
                                python3 -m bandit -r src/ -f xml -o results-bandit.xml
                            '''
                        }
                        recordIssues tools: [pyLint(name:'Bandit', pattern: 'results-bandit.xml')]
                    }
                }
            }
        }
    }
}