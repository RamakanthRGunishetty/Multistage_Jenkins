pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "jenkins/jenkins:lts"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
                script {
                    sh 'python3 -m venv ${WORKSPACE}/.venv'
                    sh '''
                    . ${WORKSPACE}/.venv/bin/activate
                    pip3 install -r ${WORKSPACE}/requirements.txt
                    '''
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    sh '''
                    . ${WORKSPACE}/.venv/bin/activate
                    python3 -m unittest discover -s .
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh '''
                    docker build -t ${DOCKER_IMAGE} ${WORKSPACE}
                    '''
                }
            }
        }
        stage('Deploy Docker Container') {
            steps {
                echo 'Deploying Docker container...'
                script {
                    // Stop and remove existing container if running
                    sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${DOCKER_IMAGE}
                    '''
                }
            }
        }
        stage('Test Application in Docker') {
            steps {
                echo 'Testing application in Docker...'
                script {
                    sh '''
                    docker exec ${CONTAINER_NAME} python3 /app/test_app.py
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
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
