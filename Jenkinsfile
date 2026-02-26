pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                # Changed path from CC_LAB-6/backend to .
                docker rmi -f backend-app || true
                docker build -t backend-app .
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create app-network || true
                docker rm -f backend1 backend2 || true
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                # Added delay to ensure backends are up before NGINX tries to find them
                sleep 3
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx
                
                # Added delay to ensure NGINX container is ready for config copy
                sleep 2

                # Changed path from CC_LAB-6/nginx/default.conf to default.conf
                docker cp default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
