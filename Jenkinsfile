pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: 084375558715.dkr.ecr.us-east-1.amazonaws.com/custom-kaniko
    command:
    - sleep
    args:
    - 9999999
    env:
    - name: AWS_REGION
      value: us-east-1
    volumeMounts:
      - name: aws-secret
        mountPath: /kaniko/.docker      
  - name: kubectl
    image: 084375558715.dkr.ecr.us-east-1.amazonaws.com/custom-kubectl
    command:
    - cat
    tty: true      
  volumes:
  - name: aws-secret
    projected:
         sources:
           - secret:
               name: kaniko-ecr-credentials
               items:
                 - key: .dockerconfigjson
                   path: config.json    
  serviceAccountName: jenkins
"""
    }
  }

  environment {
    IMAGE_NAME = '084375558715.dkr.ecr.us-east-1.amazonaws.com/my-app'
    IMAGE_TAG = 'latest'
    AWS_ACCOUNT_ID = '084375558715'
    AWS_REGION = 'us-east-1'
  }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Satish9422/my-app.git' 
            }
        }

        stage('Build & Push with Kaniko') {
          steps {
            container('kaniko') {
              sh """
                /kaniko/executor --dockerfile=Dockerfile --context=`pwd` --destination=084375558715.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
              """
            }
          }
    }


        stage('Deploy to Green') {
            steps {
              container('kubectl'){

                sh 'kubectl apply -f k8s/blue-deployment.yaml'
              
              }
            }
        }

        stage('Rollout Check'){
          steps{
            container('kubectl'){
              script {
                def status = sh(
                  script: "kubectl rollout status deployment/myapp-green --timeout=60s",
                  returnStatus: true
                )
                if (status != 0){
                  error("Deployment rollout failed. Stopping pipeline.")
                } else {
                  echo "Deployment rollout Successful"
                }
              }
            }
          }
        }

        stage('Switch Traffic') {
          when {
            expression {
              currentBuild.currentResult == 'SUCCESS'
            }
          }
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
