pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/AbhishekRangra/index_html.git'
        GIT_BRANCH = 'user'
        GIT_USERNAME = 'AbhishekRangra'
        GIT_EMAIL = 'abhishekrangra@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO}"
            }
        }

        stage('Generate Tag') {
            steps {
                script {
                    def lastTagFile = 'last_deployment_tag.txt'
                    def newTag = 'v1'

                    if (fileExists(lastTagFile)) {
                        def lastTag = readFile(lastTagFile).trim()
                        def tagNum = lastTag.replace('v', '').toInteger() + 1
                        newTag = "v${tagNum}"
                    }

                    writeFile file: lastTagFile, text: newTag
                    env.IMAGE_TAG = newTag
                    echo "Generated tag: ${newTag}"
                }
            }
        }

        stage('Update values.yaml') {
            steps {
                sh 'sed -i "s/^  tag: .*/  tag: \\"${IMAGE_TAG}\\"/" mychart/values.yaml'
            }
        }

        stage('Git Commit and Push') {
            steps {
                sh '''
                    git config user.email "${GIT_EMAIL}"
                    git config user.name "${GIT_USERNAME}"
                    git add mychart/values.yaml last_deployment_tag.txt
                    git commit -m "chore: bump image tag to ${IMAGE_TAG}" || echo "No changes to commit"
                    git push origin HEAD:${GIT_BRANCH}
                '''
            }
        }

        stage('Trigger Deployment Pipeline') {
            steps {
                build job: 'index_html_2'
            }
        }
    }
}
