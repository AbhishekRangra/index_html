pipeline {
    agent any
    environment {
        TAG_FILE = 'last_deployment_tag.txt' // File to store the version tag
        VALUES_YAML_PATH = 'mychart/values.yaml' // Path to the values.yaml file
    }
    stages {
        stage('Assign Image Tag and Update values.yaml') {
            steps {
                script {
                    // Default version to 1 if no tag file exists
                    def tag = 'v1'

                    if (fileExists(TAG_FILE)) {
                        // Read the tag file to get the last used version
                        def lastTag = readFile(TAG_FILE).trim()
                        def versionNumber = lastTag.replace('v', '') as int // Extract the version number and convert it to int
                        versionNumber++ // Increment the version number
                        tag = "v${versionNumber}" // Create new tag with the incremented version number
                    } else {
                        // If no tag file exists, initialize with v1
                        tag = 'v1'
                    }

                    // Store the new tag in the tag file to track the next deployment
                    writeFile(file: TAG_FILE, text: tag)

                    // Assign the tag to the environment variable
                    echo "Assigning tag: $tag"
                    env.DOCKER_TAG = tag

                    // Update values.yaml with the latest tag
                    echo "Updating values.yaml with tag: $tag"
                    sh """
                        sed -i 's/tag: ".*"/tag: "$DOCKER_TAG"/g' $VALUES_YAML_PATH
                        git add $VALUES_YAML_PATH
                        git commit -m "Updated Helm chart image tag to $DOCKER_TAG"
                        git push origin HEAD:user
                    """
                }
            }
        }
    }
}
