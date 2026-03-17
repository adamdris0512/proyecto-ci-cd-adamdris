pipeline {
    agent any

    environment {
        DOCKER_USER = 'adamsh04'
        IMAGE_NAME = 'python-app'
        DOCKER_CREDS_ID = 'dockerhub-creds'
        // Ruta a la configuración de Kubernetes para el usuario Jenkins
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/adamdris0512/proyecto-ci-cd-adamdris.git'
            }
        }

        stage('Test') {
            steps {
                // Instalamos dependencias y ejecutamos los tests de Python
                sh 'pip install flask pytest --break-system-packages'
                sh 'python3 -m pytest test_app.py'
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
                // Corregido: Usamos usernamePassword para que coincida con tu credencial de Jenkins
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo \$PASS | docker login -u \$USER --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                // Despliegue en Minikube usando la configuración de Jenkins
                sh "kubectl apply -f deployment.yaml --validate=false"
                sh "kubectl apply -f service.yaml --validate=false"
                sh "kubectl rollout restart deployment/${IMAGE_NAME}"
            }
        }
    }

    post {
        success {
            echo '¡Éxito! Aplicación desplegada en Kubernetes.'
        }
        failure {
            echo 'Algo ha fallado. Revisa los logs de la etapa anterior.'
        }
    }
}
