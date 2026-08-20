pipeline {
    agent any

    tools {
        jdk 'JDK 21'
        maven 'Maven 3'
    }

    environment {
        BACKEND_IMAGE = "shanmu12/fundoo-backend"
        TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $BACKEND_IMAGE:$TAG .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $BACKEND_IMAGE:$TAG
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    docker pull $BACKEND_IMAGE:$TAG

                    docker stop fundoo-backend || true
                    docker rm fundoo-backend || true

                    docker run -d \
                      --name fundoo-backend \
                      --network fundoo-network \
                      -p 8080:8080 \
                      -e SPRING_PROFILES_ACTIVE=dev \
                      --restart unless-stopped \
                      $BACKEND_IMAGE:$TAG
                '''
            }
        }
    }

    post {
        success {
            echo 'Backend Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }

        always {
            cleanWs()
        }
    }
}