pipeline{
    agent any 
       environment{
         SONAR_TOKEN = "f02c771f806bdddc47eb3d6b49a46510e1ff7191"
         AWS_REGION = "ap-south-1"
         DOCKER_IMAGE ="${ECR_REGISTRY}/ecr/repo"
         ECR_REGISTRY = "355487203430.dkr.ecr.ap-south-1.amazonaws.com"
         DOCKER_IMAGE_TAG= "${BUILD_NUMBER}"
}

        stages{
            stage('checkout scm'){
                steps{
                   git branch: 'main', url: 'https://github.com/srikanth1798/spring-petclinic.git'
                   }
            }
            stage('Maven and Sonar Stage') {
          steps {
              withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                  withSonarQubeEnv('SONARQUBE') {
                      sh '''
                         mvn package sonar:sonar \
                         -Dsonar.projectKey=veena195_spring-petclinic \
                         -Dsonar.organization=veena195 \
                         -Dsonar.host.url=https://sonarcloud.io \
                         -Dsonar.login=$SONAR_TOKEN
                     '''
               }
            }
         }
      }
       stage('image build stage'){
          steps{
              sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} .'
          }
      }
      stage('login to ECR and SCAN and push the image to ECR'){
          steps{
              withCredentials([usernamePassword(credentialsId: 'docker-login', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]){
                  sh '''
                    export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                    export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                    export AWS_DEFAULT_REGION=${AWS_REGION}
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker push ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}
                    '''
              }
          }
      }
      stage('Deploy to EKS'){
          steps{
              withCredentials([usernamePassword(credentialsId: 'docker-login', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]){
                  sh '''
                  export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                  export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                  export AWS_DEFAULT_REGION=${AWS_REGION}
                  aws eks update-kubeconfig --name ekss --region ${AWS_DEFAULT_REGION}
                  kubectl run pod1 --image ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}
                  '''
              }
          }
      }
   }
}