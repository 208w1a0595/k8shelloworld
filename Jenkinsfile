
pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = 'gowri5877'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/pythonimg"
        DOCKERHUB_TOKEN = credentials('docker-hub-credential')
        SONARQUBE_TOKEN = credentials('sonarqube-credential')
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
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {  // Use the same name as configured in Manage Jenkins
                    sh '''
                        . venv/bin/activate
                        pip install sonar-scanner  # Optional, but helps with coverage for python
                        sonar-scanner \
                        -Dsonar.projectKey=k8shelloworld \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=https://54.225.124.9:9000 \
                        -Dsonar.login=$SONARQUBE_TOKEN
                    '''
                }
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
