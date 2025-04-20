pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true      
  volumes:
    - name: kaniko-secret
      projected:
        sources:
          - secret:
              name: regcred
              items:
                - key: .dockerconfigjson
                  path: config.json
"""
    }
  }

  environment {
    IMAGE_NAME = 'satish680/my-app'
    IMAGE_TAG = 'latest'
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
                  --context=`pwd` \
                  --destination=${IMAGE_NAME}:${IMAGE_TAG}
              '''
            }
          }
    }


        stage('Deploy to Green') {
            steps {
              container('kaniko'){
                sh 'kubectl apply -f k8s/green-deployment.yaml'
                sh 'kubectl rollout status deployment/myapp-green'
              }
            }
        }

        stage('Switch Traffic') {
            steps {
            container('kubectl') {  
                sh 'kubectl patch service myapp-service -p \'{"spec":{"selector":{"app":"myapp-green"}}}\''
            }
          }
        }

        stage('Cleanup Blue') {
            steps {
            container('kubectl') {  
                sh 'kubectl delete deployment myapp-blue'
            }
          }  
        }
    }
}
