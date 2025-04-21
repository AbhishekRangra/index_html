pipeline {
    agent any

    environment {
        IMAGE_NAME = 'abhishekrangra/index_html'  // Correct repo name
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
                    docker run -d --name index_html-container -p 3000:3000 $IMAGE_NAME:$DOCKER_TAG
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

pipeline {
    agent any

    environment {
        DOCKER_TAG = "v1"
    }

    stages {
        stage('Clone Repo') {
            steps {
                checkout scm
            }
        }

        stage('Update Git Repo Image Tag') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        withCredentials([usernamePassword(
                            credentialsId: 'github',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )]) {
                            sh '''
                                git config user.email "abhishekrangra@gmail.com"
                                git config user.name "AbhishekRangra"

                                # Make your update in the repository (e.g., updating a config file)
                                sed -i 's/tag: ".*"/tag: "'"$DOCKER_TAG"'"/g' mychart/values.yaml
                                git add mychart/values.yaml
                                git commit -m "Updated image tag to $DOCKER_TAG"
                                git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/$GIT_USERNAME/index_html.git HEAD:main
                            '''
                        }
                    }
                }
            }
        }
    }
}
