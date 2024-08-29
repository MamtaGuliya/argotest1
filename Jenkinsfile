pipeline {
    agent any

    environment {
        PROJECT_ID = 'onecom-operations'
        REGISTRY_LOCATION = 'europe-west2'
        REGISTRY_NAME = 'git-repo' // Artifact Registry repository name
        IMAGE_TAG = "${env.BUILD_ID}" // Tag image with Jenkins build ID
        IMAGE_NAME = 'test-image2' // Define your Docker image name
        GOOGLE_CLOUD_KEYFILE_JSON = credentials('gcp-key')
    }

    stages {
        stage('Pull from GitHub Repo') {
            steps {
                git branch: 'main', credentialsId: 'GitHub_Auth', url: 'https://github.com/MamtaGuliya/argotest1.git'
            }
        }

        stage('Authenticate with Google Cloud') {
            steps {
                script {
                    echo "Authenticating with Google Cloud..."
                    withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}'
                        sh 'gcloud config set project ${PROJECT_ID}'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    def IMAGE_URI = "${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh 'ls -la'
                    sh 'cat Dockerfile || echo "No Dockerfile found"'
                    sh "docker build -t ${IMAGE_URI} ."
                }
            }
        }

        stage('Push Docker Image to GCP Artifact Registry') {
            steps {
                script {
                    echo "Pushing Docker image to GCP Artifact Registry..."
                    def IMAGE_URI = "${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh 'gcloud auth configure-docker ${REGISTRY_LOCATION}-docker.pkg.dev'
                    sh "docker push ${IMAGE_URI}"
                }
            }
        }
        
<<<<<<< HEAD
    }    
            post {
=======
        stage('Update Image Tag in Deployment Manifest') {
            steps {
                script {
                    // Update the image tag in the deployment.yaml file
                    echo 'Updating Image TAG in deployment.yaml'
                    sh 'sed -i "s|image: .*|image: ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}|g" manifest/deployment.yaml'

                    // Git configuration
                    echo 'Configuring Git'
                    sh 'git config --global user.email "mamta.guliya@onecom.co.uk"'
                    sh 'git config --global user.name "Mamta Guliya"'

                    // Commit and push the changes
                    sh 'git add manifest/deployment.yaml'
                    sh 'git commit -m "Update Image tag in deployment.yaml to ${IMAGE_TAG}"'

                    // Push to GitHub using the credentials
                    withCredentials([usernamePassword(credentialsId: 'github_token', usernamevariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]){
                        sh 'git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/MamtaGuliya/argotest1/manifest.git main'
                    }
                    
                }
            }
        }
    }

    post {
>>>>>>> 0058d1a55654342020029f6491b5368a124ba524
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
