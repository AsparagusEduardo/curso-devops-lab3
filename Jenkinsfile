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
                stage("Generar versión") {
                    steps {
                        script {
                            env.APP_SEMANTIC_VERSION = sh(
                                script: 'npm pkg get version | tr -d \'"\'',
                                returnStdout: true
                            ).trim()
                            echo "Versión: ${env.APP_SEMANTIC_VERSION}"
                        }
                    }
                }
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
                stage("Testeo con covertura") {
                    steps {
                        sh "npm run test:cov"
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
                    if (!env.APP_SEMANTIC_VERSION?.trim()) {
                        error("APP_SEMANTIC_VERSION no definida en el stage de anterior")
                    }
                    docker.withRegistry("https://index.docker.io/v1/","credencial-dh"){
                        sh "docker tag curso-devops-lab3 asparaguseduardo/curso-devops-lab3:latest"
                        sh "docker tag curso-devops-lab3 asparaguseduardo/curso-devops-lab3:${env.BUILD_NUMBER}"
                        sh "docker tag curso-devops-lab3 asparaguseduardo/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                        sh "docker push asparaguseduardo/curso-devops-lab3:latest"
                        sh "docker push asparaguseduardo/curso-devops-lab3:${env.BUILD_NUMBER}"
                        sh "docker push asparaguseduardo/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                    }

                    docker.withRegistry("https://ghcr.io","credencial-gh"){
                        sh "docker tag curso-devops-lab3 ghcr.io/asparaguseduardo/curso-devops-lab3:latest"
                        sh "docker tag curso-devops-lab3 ghcr.io/asparaguseduardo/curso-devops-lab3:${env.BUILD_NUMBER}"
                        sh "docker tag curso-devops-lab3 ghcr.io/asparaguseduardo/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                        sh "docker push ghcr.io/asparaguseduardo/curso-devops-lab3:latest"
                        sh "docker push ghcr.io/asparaguseduardo/curso-devops-lab3:${env.BUILD_NUMBER}"
                        sh "docker push ghcr.io/asparaguseduardo/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                    }
                }
            }
        }
    }
}