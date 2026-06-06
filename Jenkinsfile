pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                // Clonamos repositorio
                sh "git clone --branch master https://github.com/unir-mgonz/todo-list-aws.git"
            }
        }
    }
}