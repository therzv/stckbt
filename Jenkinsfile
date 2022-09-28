pipeline {
    agent any
    environment {
        CI                      = "true"
        REGISTRY_URL            = "registry.stockbit.co.id"
        SERVICE_NAME            = "${JOB_NAME.split('/')[0]}"
        OVERRIDE_SERVICE_NAME   = "rest-go"
        IMAGE_TAG               = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        DOCKER_PATH             = "rest-go-code/Dockerfile"
        HELM_PATH               = "helm_chart/${OVERRIDE_SERVICE_NAME}"
        HELM_NAMESPACE_DEV      = "dev"
        HELM_NAMESPACE_PROD     = "prod"
        IMAGE_NAME              = "${REGISTRY_URL}/${OVERRIDE_SERVICE_NAME}"
        CLUSTER_CONTEXT         = "Stockbit-K8S"
        TERRAFORM_SCRIPT        = "terraform"
        }
 }

  stages {

           stage ('Start') {
                 steps {
                    slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})  Commit Hash : ${env.GIT_COMMIT}")

                  }
              } 


            stage('Static Analysis') {
                  steps{
                     sh '''
                        echo "SonarQube Code Static Analysis"
                     '''
                   }
               }

            stage('Build and Deploy Branch Development') {

                       when {
                         expression {
                           return env.GIT_BRANCH == "origin/development"
                         }
                       } 


                       steps {

                           sh '''

                              # Docker build and push
                              docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${DOCKER_PATH} .
                              docker push  ${IMAGE_NAME}:${IMAGE_TAG}

                              # Deploy to EKS
                              kubectl config use-context ${CLUSTER_CONTEXT}
                             
                              echo "Helm install REST"
                              helm upgrade --install ${OVERRIDE_SERVICE_NAME} -f ${HELM_PATH}/values-${HELM_NAMESPACE_DEV}.yaml \
                                  --set app.imagetag=${IMAGE_TAG} \
                                  -n ${HELM_NAMESPACE_DEV} $HELM_PATH/                         
                              echo "Done"
                             
                              kubectl rollout status deployment/${OVERRIDE_SERVICE_NAME} -n ${HELM_NAMESPACE_DEV} --timeout=300s
                             
                              # Cleanup Docker Image
                              docker rmi  ${IMAGE_NAME}:${IMAGE_TAG} -f ${DOCKER_PATH} -f
                              '''
                              cleanWs ()
                       }
            }


            stage('Build and Deploy Branch Main') {

                       when {
                         expression {
                           return env.GIT_BRANCH == "origin/main"
                         }
                       } 

                       steps {

                           sh '''

                              # Docker build and push
                              docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${DOCKER_PATH} .
                              docker push  ${IMAGE_NAME}:${IMAGE_TAG}

                              # Deploy to EKS
                              kubectl config use-context ${CLUSTER_CONTEXT}
                             
                              echo "Helm install REST"
                              helm upgrade --install ${OVERRIDE_SERVICE_NAME} -f ${HELM_PATH}/values-${HELM_NAMESPACE_PROD}.yaml \
                                  --set app.imagetag=${IMAGE_TAG} \
                                  -n ${HELM_NAMESPACE_PROD} $HELM_PATH/                         
                              echo "Done"
                             
                              kubectl config use-context ${CLUSTER_CONTEXT}
                              kubectl rollout status deployment/${OVERRIDE_SERVICE_NAME} -n ${HELM_NAMESPACE_PROD} --timeout=300s
                             
                              # Cleanup Docker Image
                              docker rmi  ${IMAGE_NAME}:${IMAGE_TAG} -f ${DOCKER_PATH} -f
                              '''
                              cleanWs ()
                       
                       }
            }

    post {

        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) Commit Hash : ${env.GIT_COMMIT}")
        }

        unstable {
            slackSend (color: '#F5DE14', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) Commit Hash : ${env.GIT_COMMIT}")   
        }
  
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) Commit Hash : ${env.GIT_COMMIT} ")
        }

        aborted {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})  Commit Hash : ${env.GIT_COMMIT}")
     
        }

    }

}