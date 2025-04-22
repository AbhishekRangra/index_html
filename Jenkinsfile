pipeline {
    agent any

    environment {
        IMAGE_NAME = 'abhishekrangra/index_html'
        DOCKER_TAG = 'v1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code...'
                checkout([
                    $class: 'GitSCM',
                    userRemoteConfigs: [[
                        url: 'https://github.com/AbhishekRangra/index_html.git',
                        credentialsId: 'github'
                    ]],
                    branches: [[name: '*/main']]
                ])
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging into Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t $IMAGE_NAME:$DOCKER_TAG .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                sh '''
                    docker push $IMAGE_NAME:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy Docker Container') {
            steps {
                echo 'Deploying Docker container...'
                sh '''
                    docker rm -f index_html-container || true
                    
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Docker image built, pushed, and deployed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check error logs.'
        }
    }
}
