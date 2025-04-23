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
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  - name: kubectl
    image: satish680/custom-kubectl
    command:
    - cat
    tty: true      
  serviceAccountName: jenkins-agent  
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
    // KUBECONFIG = "${WORKSPACE}/kubeconfig"
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
                  --dockerfile=Dockerfile \
                  --context=`pwd` \
                  --destination=${IMAGE_NAME}:${IMAGE_TAG}
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
