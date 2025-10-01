pipeline {
    agent any
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
            }
        }
        stage('Docker login'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}'
                }  
            }
        }
        stage('Build Backend image and push to Docker Hub'){
            steps{
               sh '''
                    cd backend
                    docker build -t r25gajengi/easy_backend:v1 .
                    docker push r25gajengi/easy_backend:v1
                    '''
            }
        }
        stage('Run Backend container'){
            steps{
               withCredentials([string(credentialsId: 'database_url', variable: 'SPRING_DATASOURCE_URL'), string(credentialsId: 'database_user', variable: 'SPRING_DATASOURCE_USERNAME'), string(credentialsId: 'database_password', variable: 'SPRING_DATASOURCE_PASSWORD')]) {
                sh '''
                cd backend
                docker rm -f backend || true
                docker run --name backend -d \
                  -p 8081:8081 \
                  -e SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL} \
                  -e SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME} \
                  -e SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD} \
                  r25gajengi/easy_backend:v1
                '''
                }
            }
        }
        stage('Build Frontend image adn Push to Docker Hub'){
            steps{
                withCredentials([string(credentialsId: 'backend_api', variable: 'VITE_API_URL')]) {
                    sh '''
                    cd frontend
                    echo "VITE_API_URL = ${VITE_API_URL}" > .env
                    docker build -t r25gajengi/easy_frontend:v1 .
                    docker push r25gajengi/easy_frontend:v1
                    '''
                }
            }
        }
        stage('Run frontend container'){
            steps{
                sh '''
                cd frontend
                docker rm -f frontend || true
                docker run --name frontend -d \
                  -p 80:80 \
                  r25gajengi/easy_frontend:v1
                '''
            }
        }
    }
}

