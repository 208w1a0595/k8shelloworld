pipeline {
    agent any
     environment {
    DOCKERHUB_TOKEN = credentials('docker-hub-credential')
  }
    stages {
	
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    if (env.BRANCH_NAME.startsWith('feature/')) {
                        echo "Running tests on feature branch"
                        sh 'mvn clean install'
                        sh 'mvn test'
                    } else if (env.BRANCH_NAME == 'main') {
                        echo "Building, testing, and pushing Docker image on development branch"
                        sh 'mvn clean install'
                        sh 'mvn test'
                         sh "echo ${DOCKERHUB_TOKEN} | docker login -u gowri5877 --password-stdin"
                            sh 'docker build -t gowri5877/pythonimg .'
                            sh 'docker push gowri5877/pythonimg'
                        }
                    }
                }
            }
	stage('Deployment'){
		steps{

			sh 'kubectl apply -f deployment.yaml'
			sh 'kubectl apply -f service.yaml'
		
		}
        }
        }
	
    }
