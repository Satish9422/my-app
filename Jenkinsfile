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
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      command:
        - cat
      tty: true
      volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker/
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
"""
      defaultContainer 'kaniko'
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

        // stage('Build & Push') {
        //     steps {
        //       container('docker'){
        //        sh 'docker build -t my-app:latest .'
        //        sh 'docker login' 
        //        sh 'docker tag my-app:latest satish680/my-app:latest'
        //        sh 'docker push satish680/my-app:latest'
        //       }
        //     }
        // }

        stage('Build & Push with Kaniko') {
         steps {
          container('kaniko') {
           sh '''
            /kaniko/executor \
              --dockerfile=Dockerfile \
              --context=dir://workspace \
              --destination=${IMAGE_NAME}
           '''
        }
      }
    }


        stage('Deploy to Green') {
            steps {
              container('kubectl'){
                sh 'kubectl apply -f k8s/green-deployment.yaml'
                sh 'kubectl rollout status deployment/myapp-green'
              }
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
