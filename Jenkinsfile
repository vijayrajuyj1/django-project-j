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
                    sh '''
                        sudo apt update
                        sudo apt install python3-venv -y
                        python3 -m venv venv
                        . venv/bin/activate  # Activate the virtual environment
                        python3 -m pip install Django==5.1.2  # No --user flag needed
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
                    nohup python3 manage.py runserver 0.0.0.0:8000 &
                '''
            }
        }
        stage('Static Code Analysis') {
            steps {
                script {
                    // Use the SonarScanner plugin instead of calling sonar-scanner directly
                    withSonarQubeEnv('sonarqube') { // Use the name you provided in Jenkins for your SonarQube server
                        sh '''
                            . venv/bin/activate  # Activate the virtual environment
                            sonar-scanner \
                                -Dsonar.projectKey=your_project_key \
                                -Dsonar.projectName="Your Project Name" \
                                -Dsonar.projectVersion=${BUILD_NUMBER} \
                                -Dsonar.sourceEncoding=UTF-8 \
                                -Dsonar.sources=.
                        '''
                    }
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
                GIT_REPO_NAME = "django-todo-j"
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
