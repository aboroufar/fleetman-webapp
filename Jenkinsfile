pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     SERVICE_NAME = "fleetman-webapp"
     REPOSITORY_TAG="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
   }

   stages {
          
      stage('Preparation') {
         steps {
           
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
            sh '''mvn clean package'''
         }
      }

      stage('Build and Push Image') {
         steps {
            
           sh 'docker image build -t ${REPOSITORY_TAG} .'
         }
      }
      
      stage('Docker Hub Push') {
         steps {
           
            withCredentials([string(credentialsId: 'Docker-Hub', variable: 'dockerHubPwd')]) {
            sh "docker login -u apesss -p '${dockerHubPwd}'"
            sh "docker push ${REPOSITORY_TAG}"
            }   
         }
      }

      stage('Deploy to Cluster') {
          steps {
             
             sshagent (['k8s-machine']) {
                 
                 sh "sed -i 's|\${REPOSITORY_TAG}|${REPOSITORY_TAG}|g' deploy.yaml"
                 sh "scp -o StrictHostKeyChecking=no deploy.yaml root@52.54.173.163:/opt/kubernetes/${SERVICE_NAME}/"
                 script{
                     try{
                        sh "ssh root@52.54.173.163 kubectl apply -f /opt/kubernetes/${SERVICE_NAME}/deploy.yaml"
                     }catch(error){
                        sh "ssh root@52.54.173.163 kubectl apply -f /opt/kubernetes/${SERVICE_NAME}/deploy.yaml"
                      }
                
                 }
            
             }
          }
      }
   }
}
