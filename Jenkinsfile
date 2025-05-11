pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        // Multibranch Pipeline сам клонирует репо, но на всякий случай:
        checkout scm
      }
    }

    stage('Build Services') {
      steps {
        // Убедимся, что docker-compose видит файлы
        sh 'ls -R .'

        // Собираем backend и frontend
        sh 'docker-compose -f docker-compose.yml build'
      }
    }

    stage('Debug workspace') {
        steps {
            sh 'ls -R .'
        }
    }
    

    stage('Run Tests') {
      steps {
        // Поднимаем всё в фоне
        sh 'docker-compose -f docker-compose.yml up -d'

        // Пример: если в backend у вас тесты через pytest
        sh '''
          # Ждём, пока бекенд поднимется (опционально, лучше проверить таймаутами)
          sleep 5
          docker-compose -f docker-compose.yml exec backend pytest --maxfail=1 --disable-warnings -q
        '''

        // Останавливаем и удаляем контейнеры
        sh 'docker-compose -f docker-compose.yml down'
      }
    }

    stage('Push Images') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            # Предварительно в docker-compose.yml для каждого сервиса укажите image: yourrepo/backend и yourrepo/frontend
            docker-compose -f docker-compose.yml push
          '''
        }
      }
    }
  }

  post {
    always {
      // Очищаем workspace, чтобы не копились артефакты
      cleanWs()
    }
  }
}