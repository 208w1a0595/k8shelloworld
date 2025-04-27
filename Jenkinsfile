pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = 'gowri5877'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/pythonimg"
        DOCKERHUB_TOKEN = credentials('docker-hub-credential')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Build and Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                    docker build -t ${DOCKER_IMAGE} .
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
