pipeline {
    agent {
        docker {
            image 'python:3.12-slim' // Use a base image that has Python installed
            args '-u root' // Run as root to avoid permission issues
        }
    }
    stages {
        stage('Checkout') {
            steps {
                sh 'echo "Checking out the code..."'
                // Uncomment the next line to checkout the code
                // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Python venv and other dependencies
                sh '''
                    apt-get update
                    apt-get install -y python3-venv openjdk-17-jre openjdk-17-jre-headless
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Build and Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    python manage.py runserver &
                    sleep 5 # Give the server time to start
                    # You may want to run tests or perform additional commands here
                '''
            }
        }

        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://54.172.151.211:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        . venv/bin/activate
                        npx sonar-scanner -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}
                    '''
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
                    sh '''
                        docker build -t ${DOCKER_IMAGE} .
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Update Values Tag in Helm') {
            environment {
                GIT_REPO_NAME = "django-project-j"
                GIT_USER_NAME = "vijayrajuyj1"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config --global user.email "vijayarajuyj1@gmail.com"
                        git config --global user.name "vijayrajuyj1"
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
