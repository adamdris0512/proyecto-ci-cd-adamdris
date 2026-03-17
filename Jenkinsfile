pipeline {
    agent any

    environment {
        // Cambia 'adamdris0512' por tu usuario real si es distinto
        DOCKER_USER = 'adamdris0512' 
        IMAGE_NAME = 'python-app'
        DOCKER_CREDS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Clone') {
            steps {
                // Descarga el código desde tu repo
                git branch: 'main', url: 'https://github.com/adamdris0512/proyecto-ci-cd-adamdris.git'
            }
        }

        stage('Test') {
            steps {
                // Instalación forzada para evitar el error de "externally-managed-environment"
                sh 'pip install flask pytest --break-system-packages'
                sh 'pytest test_app.py'
            }
        }

        stage('Build Image') {
            steps {
                // Construcción de la imagen Docker
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('DockerHub') {
            steps {
                // Login y subida a Docker Hub usando las credenciales guardadas en Jenkins
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ENV')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER_ENV --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                // Aplicación de los cambios en Minikube (Kubernetes)
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
                // Forzamos el reinicio para que descargue la imagen nueva si ya existía
                sh "kubectl rollout restart deployment/${IMAGE_NAME}"
            }
        }
    }
}
