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
            }
        }
        stage("Lint") {
            steps {
                sh "npm run lint"
            }
        }
        stage("Testeo") {
            steps {
                sh "npm run test"
            }
        }
        stage("Construcción app") {
            steps {
                sh "npm run build"
            }
        }
        stage("Construcción Imagen Docker") {
            steps {
                sh "docker build -t curso-devops-lab3 ."
            }
        }
    }
}