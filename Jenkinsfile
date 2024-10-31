pipeline {
    agent {
        label 'java-label'
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out the code..."
                // Uncomment the next line to checkout the code
                // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Update packages and install dependencies
                    sudo apt-get update
                    sudo apt install python3-venv -y
                    
                    # Create a virtual environment in the workspace
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install Django==5.1.2

                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    python manage.py runserver &
                    sleep 5 # Give the server time to start
                    # Add test commands here if needed
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
                        npx sonar-scanner -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "vijayarajult2/django-todo:${BUILD_NUMBER}"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        docker build -t ${DOCKER_IMAGE} .
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
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
                        # Update the image tag in values.yaml
                        sed -i 's/tag: .*/tag: '${BUILD_NUMBER}'/g' helm/demo/values.yaml
                        git add helm/demo/values.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
}
