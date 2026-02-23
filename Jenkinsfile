pipeline {
  agent any
  tools {
        maven 'Maven-3.9.9'
        jdk 'JDK-17'
    }
  environment {
    DOCKER_IMAGE = 'amrutapage/simplehello'
    DOCKER_TAG = "${BUILD_NUMBER}"
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
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-credentials',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {

            powershell '''
            $pass = "$env:DOCKER_PASS"
            $pass | docker login -u $env:DOCKER_USER --password-stdin
            docker push ${env:DOCKER_IMAGE}:${env:DOCKER_TAG}
            docker push ${env:DOCKER_IMAGE}:latest
            '''
        }
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
