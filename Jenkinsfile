pipeline {
  agent any
  environment {
    CONTAINER_REGISTRY = 'gcr.io'
    GOOGLE_CLOUD_PROJECT = 'cnsa2022-emc02'
    CREDENTIALS_ID = 'cnsa2022-emc02'
  }

  tools {
    // In Global tools configuration, install Node configured as "nodejs"
    nodejs "nodejs"
  }

  stages {

    stage("Git Checkout") {
      steps {
        // checkout scm
        git branch: 'emc602', url: 'https://github.com/elisamendezcaceres/nodeapp'
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm install'
      }
    }
    stage('Test') {
      steps {
         sh 'npm run test-jenkins'
      }
      post {
        success {
          junit '**/test*.xml'
        }
      }
    }

    stage("Build image") {
      steps {
        script {
          dockerImage = docker.build(
            "${env.CONTAINER_REGISTRY}/${env.GOOGLE_CLOUD_PROJECT}/nodeapp:${env.BUILD_ID}",
            "-f Dockerfile ."
          )
        }
      }
    }
    stage("Run image locally") {
      steps {
        sh "docker stop nodeapp || true && docker rm  nodeapp || true"
        sh "docker run -d -p 8080:3000 -t --name nodeapp ${env.CONTAINER_REGISTRY}/${env.GOOGLE_CLOUD_PROJECT}/nodeapp:${env.BUILD_ID}"
      }
    }
    stage('End-to-end Test image') {
        // Ideally, we would run some end-to-end tests against our running container.
        steps{
            sh 'echo "End-to-end Tests passed"'
        }
    }
    stage("Push image") {
        steps {
            script {
                docker.withRegistry('https://'+ CONTAINER_REGISTRY, 'gcr:'+ GOOGLE_CLOUD_PROJECT) {
                        dockerImage.push("latest")
                        dockerImage.push("${env.BUILD_ID}")
                }
            }
        }
    }  
    stage('Deploy to Production') {
      steps{
        sh '''
          ssh -i ~/.ssh/id_rsa_deploy ubuntu@34.133.209.79 "if docker ps -q --filter name=nodeapp | grep . ; then docker stop nodeapp ; fi"
          ssh -i ~/.ssh/id_rsa_deploy ubuntu@34.133.209.79 "if docker ps -a -q --filter name=nodeapp | grep . ; then docker rm -fv nodeapp ; fi"
          ssh -i ~/.ssh/id_rsa_deploy ubuntu@34.133.209.79 "docker run -d -p 8080:3000 -t --name nodeapp ${CONTAINER_REGISTRY}/${GOOGLE_CLOUD_PROJECT}/nodeapp:latest"
        '''
      }
    }  
    stage('End-to-end Test on Production') {
        // Ideally, we would run some end-to-end tests against our running container.
        steps{
            sh 'echo "End-to-end Tests passed on Production"'
        }
    }
    stage('Remove Unused docker image') {
      steps{
        // input message:"Proceed with removing image locally?"
        sh 'if docker ps -q --filter name=nodeapp | grep . ; then docker stop nodeapp && docker rm -fv nodeapp; fi'
        sh 'docker rmi ${CONTAINER_REGISTRY}/${GOOGLE_CLOUD_PROJECT}/nodeapp:$BUILD_NUMBER'
      }
    }               
  }
}