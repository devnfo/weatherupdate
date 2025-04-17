pipeline {
  agent any

  environment {
    IMAGE_NAME = "weatherupdate"
    DOCKER_HUB_USER = "devjenkins_123"
    RANDOM_PORT = "8081"  // Random port chosen to avoid conflicts
  }

  stages {
    stage('Clone Repo') {
      steps {
        bat 'git config --global http.sslVerify false'
        git branch: 'main', url: 'https://github.com/devnfo/weatherupdate.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        bat """
          set DOCKER_BUILDKIT=0
          docker build -t %IMAGE_NAME% .
        """
      }
    }

    stage('Stop Previous Container') {
      steps {
        bat """
          docker stop %IMAGE_NAME% || exit 0
          docker rm %IMAGE_NAME% || exit 0
        """
      }
    }

    stage('Run Docker Container') {
      steps {
        bat "docker run -d -p %RANDOM_PORT%:80 --name %IMAGE_NAME% %IMAGE_NAME%"
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat """
            echo %PASS% | docker login -u %USER% --password-stdin
            docker tag %IMAGE_NAME% %DOCKER_HUB_USER%/%IMAGE_NAME%:latest
            docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:latest
          """
        }
      }
    }
  }

  post {
    success {
      mail to: 'your_email@gmail.com',
           subject: "✅ Jenkins Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Succeeded",
           body: "Good news!\n\nThe Jenkins build '${env.JOB_NAME} [${env.BUILD_NUMBER}]' completed successfully.\n\nCheck the build here: ${env.BUILD_URL}"
    }

    failure {
      mail to: 'your_email@gmail.com',
           subject: "❌ Jenkins Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed",
           body: "Oops!\n\nThe Jenkins build '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed.\n\nCheck the build here: ${env.BUILD_URL}"
    }
  }
}
