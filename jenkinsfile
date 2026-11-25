pipeline {
    agent { label 'build' }

    stages {
        stage('Verify Build Server') {
            steps {
                sh 'hostname'
                sh 'whoami'
                sh 'java -version'
                sh 'git --version'
                sh 'docker --version'
                sh 'docker ps'
            }
        }
    }
}
