pipeline {
    agent any
    stages {
        stage("Integración continua") {
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
            }
        }
        stage("Construcción Imagen Docker") {
            steps {
                sh "docker build -t curso-devops-lab3 ."

                script {
                    docker.withRegistry("https://index.docker.io/v1/","credencial-dh"){
                        sh "docker tag -t curso-devops-lab3 asparaguseduardo/curso-devops-lab3:latest"
                        sh "docker push asparaguseduardo/curso-devops-lab3:latest"
                    }

                    docker.withRegistry("https://ghcr.io","credencial-gh"){
                        sh "docker tag -t curso-devops-lab3 ghcr.io/asparaguseduardo/curso-devops-lab3:latest"
                        sh "docker push ghcr.io/asparaguseduardo/curso-devops-lab3:latest"
                    }
                }
            }
        }
    }
}