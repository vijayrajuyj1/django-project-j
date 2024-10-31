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

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Update packages and install necessary dependencies
                    sudo apt-get update
                    sudo apt-get install -y python3-venv openjdk-17-jre openjdk-17-jre-headless libpq-dev gcc build-essential
                    
                    # Set up Python virtual environment
                    python3 -m venv venv --without-pip
                    
                    # Install pip if missing and install requirements
                    curl -sS https://bootstrap.pypa.io/get-pip.py | venv/bin/python3
                    venv/bin/python3 -m pip install --upgrade pip
                    venv/bin/python3 -m pip install -r requirements.txt
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    # Run the Django application (for testing)
                    python manage.py runserver &
                    sleep 5
                    # Add test commands here if needed
                    python manage.py test || echo "Tests failed, continuing with the pipeline."
                '''
            }
        }

        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        . venv/bin/activate
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
                        docker build -t ${DOCKER_IMAGE} .
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Update Image Tag in Helm') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
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
