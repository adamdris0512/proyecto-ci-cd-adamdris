pipeline {
    agent any

    environment {
        DOCKER_USER = 'adamsh04'
        IMAGE_NAME = 'python-app'
        DOCKER_CREDS_ID = 'dockerhub-creds'
        // Forzamos a kubectl a usar la configuración de la carpeta de Jenkins
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
                sh 'pip install flask pytest --break-system-packages'
                sh 'python3 -m pytest test_app.py'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('DockerHub') {
            steps {
                withCredentials([string(credentialsId: "${DOCKER_CREDS_ID}", variable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                // Usamos --validate=false por si hay problemas de conexión con la API de validación
                sh "kubectl apply -f deployment.yaml --validate=false"
                sh "kubectl apply -f service.yaml --validate=false"
                sh "kubectl rollout restart deployment/${IMAGE_NAME}"
            }
        }
    }

    post {
        success {
            echo '¡Pipeline completado con éxito! La aplicación está desplegada.'
        }
        failure {
            echo 'El Pipeline ha fallado. Revisa los logs de la etapa correspondiente.'
        }
    }
}
