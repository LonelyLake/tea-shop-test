pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          // используем env.BRANCH_NAME, он всегда есть в Multibranch Pipeline
          def branch = env.BRANCH_NAME ?: sh(
            script: 'git rev-parse --abbrev-ref HEAD',
            returnStdout: true
          ).trim()
          def imageTag = "myapp:${branch}-${env.BUILD_NUMBER}"
          
          // строим образ
          sh "docker build -t ${imageTag} ."
          
          // сохраняем в переменную окружения для следующих стадий
          env.IMAGE_TAG = imageTag
        }
      }
    }

    stage('Run Tests') {
      steps {
        // запускаем тесты из только что собранного образа
        sh "docker run --rm ${env.IMAGE_TAG} ./run_tests.sh"
      }
    }

    stage('Push to Registry') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_TAG}
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