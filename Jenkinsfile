pipeline {
    agent any

    environment {
        IMAGE_NAME = "api-dotnet-analyzer"
        CONTAINER_NAME = "api-dotnet-container"
        SONAR_PROJECT_KEY = "ApiScanTest"
        SONAR_PROJECT_NAME = "ApiScanTest"
        SONAR_HOST_URL = "http://192.168.1.17:9000"
    }

    stages {
        stage('Clonar Código') {
            steps {
                git branch: 'master', url: 'https://github.com/franklincappa/test-scan-dotnet-jenkins.git'
            }
        }

        stage('Build y Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/dotnet/sdk:9.0'
                    args '-u root:root'
                }
            }
            steps {
                sh 'dotnet restore ApiScanTest.csproj'
                sh 'dotnet build ApiScanTest.csproj --no-restore'
                sh 'dotnet test ApiScanTest.csproj --no-restore --no-build'
            }
        }

        stage('Escaneo SonarQube') {
            agent {
                docker {
                    image 'mcr.microsoft.com/dotnet/sdk:9.0'
                    args '-u root:root'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        dotnet tool install --global dotnet-sonarscanner
                        export PATH="$PATH:$HOME/.dotnet/tools"
                        dotnet sonarscanner begin \
                          /k:"${SONAR_PROJECT_KEY}" \
                          /n:"${SONAR_PROJECT_NAME}" \
                          /d:sonar.host.url="${SONAR_HOST_URL}" \
                          /d:sonar.login="${SONAR_TOKEN}"

                        dotnet build ApiScanTest.csproj
                        dotnet sonarscanner end /d:sonar.login="${SONAR_TOKEN}"
                    '''
                }
            }
        }

        stage('Esperar Análisis Sonar') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Imagen Docker') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Escaneo de Vulnerabilidades (Trivy)') {
            steps {
                sh 'trivy image --severity CRITICAL,HIGH --exit-code 1 ${IMAGE_NAME}'
            }
        }

        stage('Detener Contenedor Anterior') {
            steps {
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
            }
        }

        stage('Desplegar') {
            steps {
                script {
                    sh 'docker run -d --name ${CONTAINER_NAME} -p 5006:80 ${IMAGE_NAME}'
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Fallo en el pipeline: revisar logs.'
        }
        success {
            echo '✅ Pipeline ejecutado con éxito.'
        }
    }
}
