pipeline {
    agent {
        label 'java-label'  // Replace with your agent label
    }
    environment {
        // Define environment variables if needed
        DJANGO_SETTINGS_MODULE = 'myproject.settings.production'  // Adjust this path
    }
    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }
        stage('Set Up Python Environment') {
            steps {
                script {
                    // Create and activate virtual environment
                    sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install -r requirements.txt
                    '''
                }
            }
        }
        stage('Run Migrations') {
            steps {
                script {
                    // Run Django migrations
                    sh '''
                    source venv/bin/activate
                    python manage.py migrate --noinput
                    '''
                }
            }
        }
        stage('Collect Static Files') {
            steps {
                script {
                    // Collect static files
                    sh '''
                    source venv/bin/activate
                    python manage.py collectstatic --noinput
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Add deployment commands (e.g., restart Gunicorn, deploy to server)
                    echo 'Deploying application to production server...'
                    // Example command for restarting a service
                    sh '''
                    # Add your deployment logic here, e.g., restart Gunicorn
                    sudo systemctl restart gunicorn
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment to production successful!'
        }
        failure {
            echo 'Deployment failed. Check the logs for more details.'
        }
    }
}
