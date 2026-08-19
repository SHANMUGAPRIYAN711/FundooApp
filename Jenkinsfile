pipeline {
    agent any

    tools {
        jdk 'JDK 21'
        maven 'Maven 3'
        nodejs 'NodeJS 22'
    }

    environment {
        BACKEND_IMAGE  = "shanmu12/fundoo-backend"
        FRONTEND_IMAGE = "shanmu12/fundoo-frontend"
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
                dir('Fundoo-Application') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('Fundoo-Frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('Fundoo-Application') {
                    sh 'docker build -t $BACKEND_IMAGE:$TAG .'
                }

                dir('Fundoo-Frontend') {
                    sh 'docker build -t $FRONTEND_IMAGE:$TAG .'
                }
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push $BACKEND_IMAGE:$TAG
                        docker push $FRONTEND_IMAGE:$TAG

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    docker pull $BACKEND_IMAGE:$TAG
                    docker pull $FRONTEND_IMAGE:$TAG

                    docker stop fundoo-backend || true
                    docker rm fundoo-backend || true

                    docker stop fundoo-frontend || true
                    docker rm fundoo-frontend || true

                    docker run -d \
                      --name fundoo-backend \
                      --network fundoo-network \
                      -p 8080:8080 \
                      -e SPRING_PROFILES_ACTIVE=dev \
                      --restart unless-stopped \
                      $BACKEND_IMAGE:$TAG

                    docker run -d \
                      --name fundoo-frontend \
                      --network fundoo-network \
                      -p 80:80 \
                      --restart unless-stopped \
                      $FRONTEND_IMAGE:$TAG
                '''
            }
        }
    }

    post {
        success {
            echo 'Full Stack Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }

        always {
            cleanWs()
        }
    }
}