pipeline {
  agent {
    label 'java-label'
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
      }
    }
   stage(' Install dependencies ')
      staps {
          sh ' sudo apt update '
          sh ' sudo apt install python3-venv -y '
          sh ' python3 -m venv venv '
          sh ' source venv/bin/activate '
          sh ' pip install Django==5.1.2 '

      
    stage(' Run migrations and Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh ' python manage.py makemigrations '
        sh ' $ python manage.py migrate '
        sh ' python manage.py runserver 0.0.0.0:8000 '
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://34.228.146.45/:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "vijayarajult2/petclinic1:${BUILD_NUMBER}"
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
            GIT_REPO_NAME = "petclinic-02"
            GIT_USER_NAME = "vijayrajuyj1"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "vijayarajuyj1@gmail.com"
                    git config user.name "vijayrajuyj1"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    # Update the image tag in values.yaml
                    sed -i 's/tag: .*/tag: '"${BUILD_NUMBER}"'/g' helm/demochart/values.yaml
                    git add helm/demochart/values.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
