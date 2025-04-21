pipeline {
    agent any

    environment {
        VALUES_YAML = 'mychart/values.yaml'
        TAG_FILE = 'last_deployment_tag.txt'
    }

    stages {
        stage('Generate Tag') {
            steps {
                script {
                    def newTag = 'v1'
                    if (fileExists(TAG_FILE)) {
                        def last = readFile(TAG_FILE).trim().replace('v', '') as int
                        newTag = "v${last + 1}"
                    }
                    writeFile file: TAG_FILE, text: newTag
                    env.NEW_TAG = newTag
                    echo "Generated tag: ${newTag}"
                }
            }
        }

        stage('Update values.yaml') {
            steps {
                sh """
                  sed -i 's/^  tag: .*/  tag: "${env.NEW_TAG}"/' ${VALUES_YAML}
                """
            }
        }

        stage('Git Commit and Push') {
            steps {
                sh """
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins CI"
                    git add ${VALUES_YAML} ${TAG_FILE}
                    git commit -m "chore: bump image tag to ${env.NEW_TAG}"
                    git push origin HEAD
                """
            }
        }

        stage('Trigger Deployment Pipeline') {
            steps {
                build job: 'deploy-chart', wait: false
            }
        }
    }
}
