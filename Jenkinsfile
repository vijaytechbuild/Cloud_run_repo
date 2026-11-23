pipeline {
    agent any 

    environment {
        PROJECT_ID = 'project-b6a14a58-bec5-4698-bfb'
        DOCKER_REPO = 'cloudrunlab-repo'
        DOCKER_OWNER = 'vijay0555docker'
        REGION = 'us-central1'

    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/vijaytechbuild/Cloud_run_repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_OWNER}/${DOCKER_REPO}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-password',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_OWNER}/${DOCKER_REPO}:${BUILD_NUMBER}
                        docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy to Google Cloud Run') {
            steps {
                script {
                    withCredentials([file(
                        credentialsId: 'gcp-service-account',
                        variable: 'GOOGLE_APPLICATION_CREDENTIALS'
                    )]) {

                        sh """
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project ${PROJECT_ID}
                        gcloud auth configure-docker

                        gcloud run deploy ${DOCKER_REPO} \
                            --image docker.io/${DOCKER_OWNER}/${DOCKER_REPO}:${BUILD_NUMBER} \
                            --platform managed \
                            --region ${REGION} \
                            --allow-unauthenticated \
                            --quiet
                        """
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Pipeline Failed. Check logs."
        }
    }
}