pipeline {
    agent any

    environment {
        DOCKER_USER = 'adamdris0512' 
        IMAGE_NAME = 'python-app'
        DOCKER_CREDS_ID = 'dockerhub-creds'
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
                // Usamos python3 -m para evitar el error de "command not found"
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
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ENV')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER_ENV --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
                sh "kubectl rollout restart deployment/${IMAGE_NAME}"
            }
        }
    }
}
