pipeline {
    agent any

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/NaveenSingam/devops_lab.git'
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
                docker build -t node-docker-app:${BUILD_NUMBER} .
                docker tag node-docker-app:${BUILD_NUMBER} b211626/node-docker-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push b211626/node-docker-app:${BUILD_NUMBER}
                '''
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
                minikube image load b211626/node-docker-app:${BUILD_NUMBER}
                kubectl apply -f k8/k8deployment.yml --validate=false
                '''
            }
        }

        stage('Expose Service') {
            steps {
                sh '''
                kubectl apply -f k8/k8service.yml --validate=false
                '''
            }
        }

        stage('Check Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get services
                '''
            }
        }

    }
}
