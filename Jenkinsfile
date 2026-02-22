pipeline {
  agent any
  tools {
        maven 'Maven-3.9.9'
        jdk 'JDK-17'
    }
  environment {
    DOCKER_IMAGE = 'amrutapage/simplehello'
    DOCKER_TAG = "${BUILD_NUMBER}"
    DOCKER_CREDS = credentials('dockerhub-credentials')
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/amrutapage/hello-app.git'
      }
    }

    stage('Build & Test') {
      steps {
        bat 'mvn clean test'
      }
      post {
        always {
          junit '**/target/surefire-reports/*.xml'
        }
      }
    }

    stage('Docker Build') {
      steps {
        bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
        bat "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
      }
    }

    stage('Docker Push') {
      steps {
        sh "echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin"
        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
        sh "docker push ${DOCKER_IMAGE}:latest"
      }
    }

    stage('Deploy') {
      steps {
        
        bat "docker stop springboot-app || true"
        bat "docker rm springboot-app || true"
        bat "docker run -d -p 8080:8080 --name springboot-app ${DOCKER_IMAGE}:latest"
      }
    }
  }

  post {
    success { echo 'Pipeline succeeded!' }
    failure { echo 'Pipeline failed — check logs!' }
  }
}
