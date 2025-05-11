pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "myapp:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -t $DOCKER_IMAGE .'
      }
    }
    stage('Run Tests') {
      steps {
        sh 'docker run --rm $DOCKER_IMAGE ./run_tests.sh'
      }
    }
    stage('Push to Registry') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo $PASS | docker login -u $USER --password-stdin
            docker push $DOCKER_IMAGE
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
