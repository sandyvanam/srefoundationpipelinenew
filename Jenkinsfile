pipeline {
  agent any
  environment{
     
      dockeruser = "${params.dockeruser}"
      dockerpass = "${params.dockerpass}"
  }
  
  stages {
    stage("Application Code Cloning from GitLab"){
      steps{
         git 'https://gitlab.com/sandyvanam/srefoundationpipelinenew.git'
      }
    }
    stage('Packeging the Application'){
      steps{
         sh label: '', script: 'mvn clean package'
         echo "${BUILD_URL}"
      }
    }
    stage('Static Code Analysis'){
      steps{
        echo "Stage Completed"
        
      }
    }
    stage("Building Docker Image"){
      steps{
         sh 'docker build -t spring-boot-image .'
         sh 'docker image list'
         sh 'docker tag spring-boot-image ${dockeruser}/spring-boot-java-web-service:latest'
         sh 'docker tag spring-boot-image ${dockeruser}/spring-boot-java-web-service:$BUILD_NUMBER'
      }
    }
    stage("Push Image to Docker Hub"){
      steps{
            
           sh 'docker login -u ${dockeruser} -p ${dockerpass}'
           sh 'docker push ${dockeruser}/spring-boot-java-web-service:$BUILD_NUMBER'
           sh 'docker image rm spring-boot-image:latest'
           sh 'docker image rm ${dockeruser}/spring-boot-java-web-service:latest'
           sh 'docker image rm ${dockeruser}/spring-boot-java-web-service:$BUILD_NUMBER'
         
      }
    }
    
    stage('Deploying Application in "Development" Environment'){
      steps{
          sh "sed -i 's/DOCKERUSER/${dockeruser}/g' deployment_dev.yaml"
          sh "sed -i 's/BUILD_NUMBER/$BUILD_NUMBER/g' deployment_dev.yaml"
         script{
                        sh "sudo /usr/local/bin/kubectl  apply -f deployment_dev.yaml"
         }
      }
    }
    stage('Deploying Application in "Test" Environment'){
      steps{
          sh "sed -i 's/DOCKERUSER/${dockeruser}/g' deployment_test.yaml"
          sh "sed -i 's/BUILD_NUMBER/$BUILD_NUMBER/g' deployment_test.yaml"
        script{
                        sh "sudo /usr/local/bin/kubectl apply -f deployment_test.yaml"
        }
      }
    }
    stage('Deploying Application in "Production" Environment'){
      steps{
          sh "sed -i 's/DOCKERUSER/${dockeruser}/g' deployment_production.yaml"
          sh "sed -i 's/BUILD_NUMBER/$BUILD_NUMBER/g' deployment_production.yaml"
        script{
                        sh "sudo /usr/local/bin/kubectl apply -f deployment_production.yaml"
        }
      }
    }
 }
}
