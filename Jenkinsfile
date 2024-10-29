pipeline {
    agent any
    environment {
        SONAR_URL = "http://54.172.151.211:9000"
        DOCKER_IMAGE = "vijayarajult2/django-todo:${BUILD_NUMBER}"
        GIT_REPO_NAME = "django-project-j"
        GIT_USER_NAME = "vijayrajuyj1"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code'
                // Uncomment the line below to enable Git checkout
                // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies'
                sh 'sudo apt-get update -y'
                sh 'sudo apt-get install python3-venv -y'
                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Build and Test') {
            steps {
                echo 'Listing files and running the Django server'
                sh 'ls -ltr'
                sh 'python3 manage.py runserver & sleep 5' // Running with a sleep delay for testing
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo 'Performing static code analysis with SonarQube'
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'npx sonar-scanner -Dsonar.login=${SONAR_AUTH_TOKEN} -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps {
                echo 'Building and pushing Docker image'
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Values Tag in Helm') {
            steps {
                echo 'Updating Helm chart values with new image tag'
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "vijayarajuyj1@gmail.com"
                        git config user.name "vijayrajuyj1"
                        sed -i 's/tag: .*/tag: ${BUILD_NUMBER}/' helm/demo/values.yaml
                        git add helm/demo/values.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
