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
        stage("Quality Assurance"){
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli'
                    args '--network=devops-infra_default'
                    reuseNode true
                }
            }
            stages{
                stage("validacion de codigo"){
                    steps{
                        withSonarQubeEnv('sonarqube'){
                            sh 'sonar-scanner'
                        }
                    }
                }
                stage('validacion quality gate'){
                    steps{
                        script{
                            def  qualityGate = waitForQualityGate() // esperar por el resultado del qualitygate en un endpoint de jenkins, que se gatilla desde sonar via webhook.
                            if(qualityGate.status != 'OK'){
                                error "La puerta de calidad ha fallado : ${qualityGate.status}"
                            }
                        }
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
        stage("Despliegue") {
            agent {
                docker {
                    image 'alpine/k8s:1.34.6'
                    reuseNode true
                }
            }
            steps {
                script {
                    if (!env.APP_SEMANTIC_VERSION?.trim()) {
                        error("APP_SEMANTIC_VERSION no definida en el stage de anterior")
                    }
                }

                withKubeConfig([credentialsId: 'credencial-k8']) {
                    sh "kubectl -n equezada set image deployement/curso-devops-deployment contenedor-curso-devops=asparaguseduardo/curso-devops-lab3:latest"
                }
            }
        }
    }
}