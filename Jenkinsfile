pipeline {
    agent any

    // environment {
    //     KUBECONFIG = credentials('eks-kubeconfig')  // or configure with aws eks update-kubeconfig
    // }

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

//         stage('Smoke Test') {
//             steps {
//                 script {
//             def podName = sh(script: "kubectl get pods -l app=myapp-green -o jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
//             def podIP = sh(script: "kubectl get pod ${podName} -o jsonpath='{.status.podIP}'", returnStdout: true).trim()
//             sh "curl http://${podIP}:3000/health"
// }
//             }
//         }

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
