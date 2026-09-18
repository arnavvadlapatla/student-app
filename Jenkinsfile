pipeline {
    agent any

    environment {
        IMAGE_NAME = "student-app"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "student-app-container"
        APP_PORT = "8080"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git...'
                checkout scm
            }
        }

        stage('Build & Test (Maven)') {
            steps {
                sh 'mvn -B clean package'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:8080 ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    sleep 5
                    curl --fail http://localhost:${APP_PORT}/health
                """
            }
        }
    }

    post {
        success { echo "Pipeline completed successfully. App is running on port ${APP_PORT}." }
        failure { echo 'Pipeline failed. Check the stage logs above.' }
    }
}
