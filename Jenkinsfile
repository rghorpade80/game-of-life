pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                mvn package
            }
        }
       
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
