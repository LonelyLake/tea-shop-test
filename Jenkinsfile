pipeline {
  agent any

  environment {
    // Берём имя ветки, в которой запущен билд
    GIT_BRANCH_NAME = env.BRANCH_NAME ?: sh(
      script: 'git rev-parse --abbrev-ref HEAD',
      returnStdout: true
    ).trim()
    // Формируем тег для образа
    DOCKER_IMAGE = "myapp:${GIT_BRANCH_NAME}-${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        // Именно этот шаг делает git-clone, и без него .git не создаётся
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${DOCKER_IMAGE} ."
      }
    }

    stage('Run Tests') {
      steps {
        sh "docker run --rm ${DOCKER_IMAGE} ./run_tests.sh"
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
            docker push ${DOCKER_IMAGE}
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