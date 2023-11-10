pipeline {
  agent { label 'Jenkins-Agent'}
  tools {
    jdk 'Java17'
    maven 'Maven3'
  }
  environment{
    APP_NAME = "register-app-pipe"
    RELEASE = "1.0.0"
    DOCKER_USER = "mandals0908"
    DOCKER_PASS = 'dockerhub'
    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
  }
  stages {
    stage("Cleanup Workspace"){
      steps {
        cleanWs()
        }
    }
    stage("Checkout from SCM"){
      steps{
        git branch: 'main', credentialsId: 'Github' , url: 'https://github.com/mandals0908/register-app.git'
      }
    }
    stage("Build Application"){
      steps{
        sh "mvn clean package"
      }
    }
    stage("Test Application"){
      steps{
        sh "mvn test"
      }
    }
    stage("Sonarqube Analysis"){
      steps{
        script{
          withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
          sh "mvn sonar:sonar"
          }
        }
      }
    }
    stage("Quality Gates"){
      steps{
        script{
          waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
        }
      }
    }
    stage("Build & Push Docker Image"){
      steps{
        script{
          docker.withRegistry('',DOCKER_PASS){
            docker_image = docker.build "${IMAGE_NAME}"
          }
          docker.withRegistry('',DOCKER_PASS){
            docker_image.push("${IMAGE_TAG}")
            docker_image.push('latest')
          }
        }
      }
      
    }
    stage("Trivy Scan"){
      steps{
        script{
          sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mandals0908/register-app-pipe:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
        }
      }
    }
    stage("Clean Artifacts"){
      steps{
        script{
          sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
          sh "docker rmi ${IMAGE_NAME}:latest"
        }
      }
    }
    stage("Trigger CD pipeline"){
      steps{
        script{
          sh "curl -v -k --user Sourav:${'JENKINS_API_TOKEN'} -X POST -H 'cache-control:no cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-233-132-147.ap-south-1.compute.amazonaws.com/job/Gitops-cd-pipe/buildWithParameters?token=Gitops-token'"
        }
      }
    }
  }
}
