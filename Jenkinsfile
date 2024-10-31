pipeline {
    agent {
        label 'java-label'
    }

    environment {
        DOCKER_IMAGE = "vijayarajult2/django-todo:${BUILD_NUMBER}"
        GIT_REPO_NAME = "django-project-j"
        GIT_USER_NAME = "vijayrajuyj1"
        SONAR_URL = "http://54.172.151.211:9000"  // Replace with actual SonarQube URL
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out the code..."
                checkout scm
            }
        }

        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        # Activate virtual environment if needed
                        # . venv/bin/activate
                        npx sonar-scanner \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.host.url=${SONAR_URL}
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
                        
                        # Push Docker image to registry
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Update Image Tag in Helm') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        # Configure Git
                        git config --global user.email "vijayarajuyj1@gmail.com"
                        git config --global user.name "vijayrajuyj1"
                        
                        # Update the image tag in Helm values.yaml
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
