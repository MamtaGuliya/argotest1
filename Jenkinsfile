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
        stage('Trigger ManifestUpdate') {
            steps {
                script {
                    echo "Triggering update manifest job..."
                    build job: 'updateimage', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_ID)]
                    build job: 'updateimage', parameters: [
                    string(name: 'REGISTRY_NAME', value: env.REGISTRY_NAME),
                    string(name: 'IMAGE_TAG', value: env.IMAGE_TAG),
                    string(name: 'IMAGE_NAME', value: env.IMAGE_NAME),
                    ]
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
