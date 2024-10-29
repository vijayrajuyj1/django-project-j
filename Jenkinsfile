pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        // Install Python dependencies
        sh 'sudo apt install python3-venv -y'
        sh 'pip3 install -r requirements.txt'
            }
        }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'python manage.py runserver'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://54.172.151.211:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'npx sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "vijayarajult2/django-todo:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Values tag in helm ') {
        environment {
            GIT_REPO_NAME = "django-project-j"
            GIT_USER_NAME = "vijayrajuyj1"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "vijayarajuyj1@gmail.com"
                    git config user.name "vijayrajuyj1"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    # Update the image tag in values.yaml
                    sed -i 's/tag: .*/tag: '"${BUILD_NUMBER}"'/g' helm/demo/values.yaml
                    git add helm/demo/values.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
