pipeline {
    agent any

    environment {
        IMAGE_NAME = "ashok918/node-docker-app"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/EslavathAshok123/devops_lab.git'
            }
        }

        stage('Clean Workspace') {
            steps {
                sh '''
                npm cache clean --force
                rm -rf node_modules
                rm -f package-lock.json
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }

        stage('Start Minikube') {
            steps {
                sh '''
                minikube status || minikube start --driver=docker
                kubectl config use-context minikube
                '''
            }
        }

        stage('Deployment of nodeapp') {
            steps {
                sh '''
                # Load image into minikube
                minikube image load ${IMAGE_NAME}:${BUILD_NUMBER}

                # Delete old deployment (if exists)
                kubectl delete deployment my-deployment --ignore-not-found

                # Replace IMAGE_TAG and apply
                sed "s/IMAGE_TAG/${BUILD_NUMBER}/g" k8/k8deployment.yml | kubectl apply -f - --validate=false
                '''
            }
        }

        stage('Expose Service') {
            steps {
                sh '''
                # Delete old service (if exists)
                kubectl delete service my-service --ignore-not-found

                # Apply service
                kubectl apply -f k8/k8service.yml --validate=false
                '''
            }
        }

        stage('Check Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get svc
                echo "Deployment done!"
                '''
            }
        }
    }
}
