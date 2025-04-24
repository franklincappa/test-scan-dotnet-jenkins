pipeline {
    agent any

    environment {
        IMAGE_NAME = "api-dotnet-analyzer"
        CONTAINER_NAME = "api-dotnet-container"
        SONAR_PROJECT_KEY = "dotnet-api"
        SONAR_PROJECT_NAME = "API .NET Análisis"
        SONAR_HOST_URL = "http://localhost:9000"
    }

    stages {
        stage('Clonar Código') {
            steps {
                git branch: 'master', url: 'https://github.com/franklincappa/test-scan-dotnet-jenkins.git'
            }
        }

        stage('Build y Test') {
            steps {
                sh 'dotnet restore'
                sh 'dotnet build --no-restore'
                sh 'dotnet test --no-restore --no-build'
            }
        }

        stage('Escaneo SonarQube') {
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

                        dotnet build
                        dotnet sonarscanner end /d:sonar.login="${SONAR_TOKEN}"
                    '''
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
                sh 'trivy image --severity CRITICAL,HIGH --exit-code 1 ${IMAGE_NAME} || true'
            }
        }

        stage('Desplegar') {
            steps {
                script {
                    sh 'docker stop ${CONTAINER_NAME} || true'
                    sh 'docker rm ${CONTAINER_NAME} || true'
                    sh 'docker run -d --name ${CONTAINER_NAME} -p 5006:8080 ${IMAGE_NAME}'
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
