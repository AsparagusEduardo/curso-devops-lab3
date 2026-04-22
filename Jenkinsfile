pipeline {
    agent {
        docker {
            image "node:24"
        }
    }
    stages {
        stage("Instalación de dependencias") {
            steps {
                sh "npm install"
                sh "ls -l"
                sh "hostname"
            }
        }
    }
}