pipeline {
    agent {
        label 'java-label'
    }
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                    set -e
                    if [ ! -d "venv" ]; then
                        sudo apt-get update
                        sudo apt-get install -y python3-venv openjdk-17-jre openjdk-17-jre-headless libpq-dev gcc
                        python3 -m venv venv
                    fi
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e
                    venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    python manage.py runserver &
                    sleep 5
                    # Insert additional test commands if necessary
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
