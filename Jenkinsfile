pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build with Docker-Compose') {
      steps {
        // Собираем образы по docker-compose.yml
        sh 'docker-compose -f docker-compose.yml build'
      }
    }

    stage('Run Tests') {
      steps {
        // Поднимаем сервисы, выполняем тесты, сворачиваем
        sh '''
          docker-compose -f docker-compose.yml up -d
          # Предположим, ваш основной сервис называется "tea-shop"
          docker exec tea-shop sh -c "./run_tests.sh"
          docker-compose -f docker-compose.yml down
        '''
      }
    }

    stage('Push Images (main only)') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker-compose -f docker-compose.yml push
          '''
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}