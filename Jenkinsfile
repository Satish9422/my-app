pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:latest
      imagePullPolicy: IfNotPresent
    - name: docker
      image: docker:20.10.24-dind
      command:
        - cat
      tty: true
    - name: kubectl
      image: bitnami/kubectl:latest
      command:
        - cat
      tty: true
"""
      defaultContainer 'kubectl'
    }
  }
    environment {
      IMAGE_NAME = 'satish680/my-app:latest'
      // KUBECONFIG = "/var/run/secrets/kubernetes.io/serviceaccount/token"

  }

    stages {
        // stage('Checkout') {
        //     steps {
        //         git branch: 'main', url: 'https://github.com/Satish9422/my-app.git' 
        //     }
        // }

        stage('Build & Push') {
            steps {

               sh 'docker build -t my-app:latest .'
               sh 'docker login' 
               sh 'docker tag my-app:latest satish680/my-app:latest'
               sh 'docker push satish680/my-app:latest'
            }
        }

    //     stage('Build and Push with Kaniko') {
    //      steps {
    //        container('kaniko') {
    //         sh '''
    //         /kaniko/executor \
    //           --dockerfile=Dockerfile \
    //           --context=dir:///workspace \
    //           --destination=satish680/my-app:latest
    //       '''
    //     }
    //   }
    // }

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
