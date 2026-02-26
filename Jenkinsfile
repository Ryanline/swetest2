FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY . /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]




pipeline {
    agent any


    environment {
        DOCKERHUB_USER = "ryanline"
        IMAGE_NAME     = "simple-webapp"
        IMAGE_TAG      = "latest"
    }


    stages {


        stage('Checkout') {
            steps {
                git 'https://github.com/ryanline/swetest2.git'
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }


        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-pass') {
                        dockerImage.push()
                    }
                }
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/simple-webapp-deployment \
                simple-webapp=${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}


                kubectl rollout status deployment/simple-webapp-deployment
                '''
            }
        }
    }


    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
