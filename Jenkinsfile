pipeline {

    agent any

    environment {
        IMAGE_NAME = "mlops-flask-app"
        CONTAINER_NAME = "mlops-container"
        APP_PORT = "5000"
    }

    stages {

        stage('1 - Checkout') {
            steps {
                echo 'Checking out source code...'
            }
        }

        stage('2 - Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    python3 -m venv .jenkins-venv
                    . .jenkins-venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('3 - Run Tests') {
            steps {
                echo 'Running automated tests...'
                sh '''
                    . .jenkins-venv/bin/activate
                    pytest -v
                '''
            }
        }

        stage('4 - Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('5 - Deploy Container') {
            steps {
                echo 'Deploying Docker container...'
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:5000 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('6 - Health Check') {
            steps {
                echo 'Checking application health...'

                sh '''
                    sleep 5
                    curl --fail http://localhost:${APP_PORT}/health
                '''
            }
        }

        stage('7 - Deployment Information') {
            steps {
                sh '''
                    echo "========================================="
                    echo "       MLOPS DEPLOYMENT COMPLETE"
                    echo "========================================="
                    echo "Build Number : ${BUILD_NUMBER}"
                    echo "Build Status : ${currentBuild.currentResult}"
                    echo "Application  : http://localhost:${APP_PORT}"
                    echo "Health Check : PASSED"
                    echo "Docker Image : ${IMAGE_NAME}:${BUILD_NUMBER}"
                    echo "========================================="
                '''
            }
        }
    }

    post {

        success {
            script {
                def endTime = new Date().format(
                    "yyyy-MM-dd HH:mm:ss",
                    TimeZone.getDefault()
                )

                echo """
=========================================
       MLOPS DEPLOYMENT SUCCESS
=========================================
Build Number : ${BUILD_NUMBER}
Status       : SUCCESS

Start Time   : ${new Date(currentBuild.startTimeInMillis)}
End Time     : ${endTime}

Duration     : ${currentBuild.durationString}

Docker       : SUCCESS
Tests        : SUCCESS
Health Check : SUCCESS
Deployment   : SUCCESS

Application:
http://localhost:${APP_PORT}

=========================================
       ALL WORK COMPLETED
=========================================
"""
            }
        }

        failure {
            script {
                def endTime = new Date().format(
                    "yyyy-MM-dd HH:mm:ss",
                    TimeZone.getDefault()
                )

                echo """
=========================================
       MLOPS DEPLOYMENT FAILED
=========================================
Build Number : ${BUILD_NUMBER}
Status       : FAILED

Start Time   : ${new Date(currentBuild.startTimeInMillis)}
End Time     : ${endTime}

Duration     : ${currentBuild.durationString}

Please check Jenkins Console Output.

=========================================
"""
            }
        }

        always {
            echo "Pipeline execution finished."
        }
    }
}
