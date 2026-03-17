pipeline {
    agent any

    environment {
        // SUSTITUYE con tu usuario real de Docker Hub
        DOCKER_USER = 'adamdris0512' 
        IMAGE_NAME = 'python-app'
        // Este ID debe coincidir con el que creaste en Jenkins -> Credentials
        DOCKER_CREDS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                // Instalamos dependencias y corremos pytest
                sh 'pip install flask pytest'
                sh 'pytest test_app.py'
            }
        }

        stage('Build Image') {
            steps {
                // Construimos la imagen usando el Dockerfile de la carpeta
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('DockerHub') {
            steps {
                // Iniciamos sesión y subimos la imagen
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ENV')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER_ENV --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                // Desplegamos en Kubernetes (Minikube)
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
                // Forzamos el reinicio para que pille la imagen nueva si ya existía
                sh "kubectl rollout restart deployment/${IMAGE_NAME}"
            }
        }
    }
}
