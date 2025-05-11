pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build with Docker Compose') {
      steps {
        // Собираем все сервисы, описанные в docker-compose.yml
        sh 'docker compose -f docker-compose.yml build'
      }
    }

    stage('Run Tests') {
      steps {
        // Поднимаем контейнер(ы) в фоне, ждём доступности, запускаем тесты
        sh '''
          docker compose -f docker-compose.yml up -d
          # здесь ваш скрипт тестов, например:
          docker exec tea-shop sh -c "./run_tests.sh"
          docker compose -f docker-compose.yml down
        '''
      }
    }

    stage('Push Image (main only)') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            # docker compose push, если в yml прописан image & push
            docker compose -f docker-compose.yml push
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