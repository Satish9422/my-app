pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: satish680/custom-kaniko:latest
    command:
    - sleep
    args:
    - 9999999
    env:
    - name: AWS_REGION
      value: us-east-1
    volumeMounts:
      - name: aws-secret
        mountPath: /kaniko/.aws
  volumes:
  - name: aws-secret
    secret:
      secretName: aws-ecr-credentials      
  - name: kubectl
    image: satish680/custom-kubectl
    command:
    - cat
    tty: true      
  serviceAccountName: jenkins 
  volumes:
    - name: aws-secret
      secret:
        secretName: kaniko-ecr-credentials
"""
    }
  }

  environment {
    IMAGE_REPO_NAME = 'my-app'
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
              sh '''
                
                /kaniko/executor \
                  --
                  --dockerfile=Dockerfile \
                  --context=`pwd` \
                  --destination=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG} \
                  --insecure \
                  --skip-tls-verify
              '''
            }
          }
    }


        stage('Deploy to Green') {
            steps {
              container('kubectl'){

                sh 'kubectl apply -f k8s/green-deployment.yaml'
              
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

        // stage('Cleanup Blue') {
        //     steps {
        //     container('kubectl') {  
        //         sh 'kubectl delete deployment myapp-blue'
        //     }
        //   }  
        // }
    }
}
