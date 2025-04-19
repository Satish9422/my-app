pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Satish9422/my-app.git'
            }
        }

        stage('Build & Push') {
            steps {

               sh 'docker build -t my-app:latest .'
               sh 'docker login' 
               sh 'docker tag my-app:latest satish680/my-app:latest'
               sh 'docker push satish680/my-app:latest'
            }
        }

        stage('Deploy to Green') {
            steps {
                sh 'kubectl apply -f k8s/green-deployment.yaml'
                sh 'kubectl rollout status deployment/myapp-green'
            }
        }

        stage('Switch Traffic') {
            steps {
                sh 'kubectl patch service myapp-service -p \'{"spec":{"selector":{"app":"myapp-green"}}}\''
            }
        }

        stage('Cleanup Blue') {
            steps {
                sh 'kubectl delete deployment myapp-blue'
            }
        }
    }
}
