pipeline {
  agent any
  environment{
    scannerHome = tool 'SonarScanner'
    JavaHome = tool 'JAVA_HOME'
  }
  stages {
    stage("Application Code Cloning from GitLab"){
      steps{
         git credentialsId: 'GIT_LAB_CREDENTIALS', url: 'https://gitlab.com/sujalmitra/docker-spring-boot-java-web-service.git'
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
        withEnv(["JAVA_HOME=$JavaHome"]){
          withSonarQubeEnv("SonarCloud"){
            sh "${tool("SonarScanner")}/bin/sonar-scanner"
          }
        }
      }
    }
    stage("Building Docker Image"){
      steps{
         sh 'docker build -t spring-boot-image .'
         sh 'docker image list'
         sh 'docker tag spring-boot-image sujal2308/spring-boot-java-web-service:latest'
         sh 'docker tag spring-boot-image sujal2308/spring-boot-java-web-service:$BUILD_NUMBER'
      }
    }
    stage("Push Image to Docker Hub"){
      steps{
         withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'PASSWORD')]){
           sh 'docker login -u sujal2308 -p $PASSWORD'
           sh 'docker push sujal2308/spring-boot-java-web-service:$BUILD_NUMBER'
           sh 'docker image rm spring-boot-image:latest'
           sh 'docker image rm sujal2308/spring-boot-java-web-service:latest'
           sh 'docker image rm sujal2308/spring-boot-java-web-service:$BUILD_NUMBER'
         }
      }
    }
    stage('Deploying Application in "Development" Environment'){
      steps{
         script{
           sh "kubectl apply -f deployment_dev.yaml"
         }
      }
    }
    stage('Deploying Application in "Test" Environment'){
      steps{
        script{
           sh "kubectl apply -f deployment_test.yaml"
        }
      }
    }
    stage('Deploying Application in "Production" Environment'){
      steps{
        script{
           sh "kubectl apply -f deployment_production.yaml"
        }
      }
    }
 }
}
