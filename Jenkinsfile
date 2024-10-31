pipeline {
    agent {
        label 'java-label'
    }
    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed'
                // Uncomment the following line to check out the repository
                // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
            }
        }
        stage('Install dependencies') {
            steps {
                script {
                    // Use bash for the shell commands
                    sh '''
                        sudo apt update
                        sudo apt install python3-venv -y
                        python3 -m venv venv
                        . venv/bin/activate  # Activate the virtual environment
                        python3 -m pip install --user Django==5.1.2  # Use python3 -m pip to install
                    '''
                }
            }
        }
        stage('Run migrations and Build and Test') {
            steps {
                sh 'ls -ltr'
                // Build the project and create a JAR file
                sh '''
                    . venv/bin/activate  # Activate the virtual environment
                    python3 manage.py makemigrations
                    python3 manage.py migrate
                    python3 manage.py runserver 0.0.0.0:8000
                '''
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://34.228.146.45:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        . venv/bin/activate  # Activate the virtual environment
                        python3 -m pip install --user mvn  # Install Maven if needed
                        mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=$SONAR_URL
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
                        . venv/bin/activate  # Activate the virtual environment
                        docker build -t ${DOCKER_IMAGE} .
                        def dockerImage = docker.image("${DOCKER_IMAGE}")
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                            dockerImage.push()
                        }
                    '''
                }
            }
        }
        stage('Update Values tag in Helm') {
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
