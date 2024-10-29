pipeline {
    agent any
    environment {
        VENV_PATH = ".venv"
        SONAR_URL = "http://54.172.151.211:9000"
        DOCKER_IMAGE = "vijayarajult2/django-todo:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
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
                echo 'Setting up Python virtual environment and installing dependencies'
                // Create and activate a virtual environment
                sh '''
                    apt install python3.12-venv
                    python3 -m venv ${VENV_PATH}
                    source ${VENV_PATH}/bin/activate
                    ${VENV_PATH}/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Build and Test') {
            steps {
                echo 'Running tests in the virtual environment'
                sh '''
                    source ${VENV_PATH}/bin/activate
                    ${VENV_PATH}/bin/python manage.py runserver &
                
                    # Add your testing command here, e.g. pytest or manage.py test
                    ${VENV_PATH}/bin/python manage.py 
                '''
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo 'Performing static code analysis with SonarQube...'
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        npx sonar-scanner -Dsonar.projectKey=django-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                echo 'Building Docker image and pushing to Docker Hub...'
                script {
                    sh '''
                        docker build -t ${DOCKER_IMAGE} .
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                            docker.image("${DOCKER_IMAGE}").push()
                        }
                    '''
                }
            }
        }

        stage('Update Values Tag in Helm') {
            steps {
                echo 'Updating Helm values with new image tag...'
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

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
