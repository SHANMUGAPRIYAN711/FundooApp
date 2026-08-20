
pipeline {
    agent any

    tools {
        maven 'Maven 3'
    }

    environment {
        JAVA_HOME = "/opt/java/openjdk"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        BACKEND_IMAGE = "shanmu12/fundoo-backend"
        TAG = "latest"
        EC2_HOST = "18.60.209.203"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Java') {
            steps {
                sh '''
                    echo "JAVA_HOME=$JAVA_HOME"
                    java -version
                    mvn -version
                '''
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
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST << 'EOF'
cd ~/fundoo

docker compose pull

docker compose stop fundoo-backend || true
docker compose rm -f fundoo-backend || true

docker compose up -d

docker ps
EOF
                    '''
                }
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