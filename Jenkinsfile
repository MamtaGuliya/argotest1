node {
    def app

    // Environment variables
    def PROJECT_ID = 'onecom-operations'
    def REGISTRY_LOCATION = 'europe-west2'
    def REGISTRY_NAME = 'git-repo'
    def IMAGE_TAG = "${env.BUILD_ID}"
    def IMAGE_NAME = 'test-image2'
    def IMAGE_URI = "${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"

    stage('Clone repository') {
        checkout scm
    }

    stage('Authenticate with Google Cloud') {
        withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
            sh 'gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}'
            sh 'gcloud config set project ${PROJECT_ID}'
        }
    }

    stage('Build image') {
        app = docker.build("${IMAGE_URI}")
    }

    stage('Test image') {
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image to GCP Artifact Registry') {
        sh 'gcloud auth configure-docker ${REGISTRY_LOCATION}-docker.pkg.dev'
        app.push("${IMAGE_TAG}")
    }
    
    stage('Trigger ManifestUpdate') {
        echo "Triggering updateManifestJob..."
        build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: IMAGE_TAG)]
    }
}