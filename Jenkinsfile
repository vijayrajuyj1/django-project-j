pipeline {
    agent {
        label 'java-label'
    }

    environment {
        SONAR_URL = "http://54.172.151.211:9000" // Replace with actual SonarQube URL
        DOCKER_IMAGE = "vijayarajult2/django-todo:${BUILD_NUMBER}"
        GIT_REPO_NAME = "django-project-j"
        GIT_USER_NAME = "vijayrajuyj1"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out the code..."
                // Uncomment the next line to checkout the code
                // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
                checkout scm
            }
        }

        stage('Create Virtual Environment and Install Dependencies') {
            steps {
                sh '''
                    # Update packages and install dependencies
                    sudo apt-get update
                    sudo apt-get install python3-venv -y
                    
                    # Create a virtual environment in the workspace
                    python3 -m venv venv

                    # Activate the virtual environment and install Django
                    . venv/bin/activate
                    pip install Django==5.1.2
                '''
            }
        }

        stage('Static Code Analysis') {
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
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        # Build Docker image
                        docker build -t ${DOCKER_IMAGE} .
                        
                        # Log in to Docker registry
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        
                        # Push Docker image to the registry
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Update Values Tag in Helm') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        # Configure Git
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

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline completed with errors, but continuing.'
        }
    }
}
