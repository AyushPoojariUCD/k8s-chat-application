pipeline {
  agent any

  environment {
    SONAR_HOME = tool "sonarqube"   // Jenkins SonarQube scanner installation name
  }

  stages {
    stage('Clean UP') {
      steps {
        cleanWs()
      }
    }

    stage('Checkout Code') {
      steps {
        git branch: 'main', // or master if your branch is master
            url: 'https://github.com/AyushPoojariUCD/k8s-chat-application.git'
      }
    }

    stage('Code Quality') {
      steps {
        withSonarQubeEnv("sonarqube") {
          sh """
            ${SONAR_HOME}/bin/sonar-scanner \
              -Dsonar.projectKey=chat-app \
              -Dsonar.projectName=chat-app \
              -Dsonar.sources=.
          """
        }
      }
    }

    stage('Build & Push Docker Images') {
      steps {
        script {
          // Login to Docker Hub (store creds in Jenkins credentials with ID 'dockerhub')
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
          }

          // Build and push backend image
          sh """
            docker build -t ayushpoojari/chatapp-backend:latest ./backend
            docker push ayushpoojari/chatapp-backend:latest
          """

          // Build and push frontend image
          sh """
            docker build -t ayushpoojari/chatapp-frontend:latest ./frontend
            docker push ayushpoojari/chatapp-frontend:latest
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          // Bring up containers
          sh 'docker compose up -d'
        }
      }
    }
  }
}
