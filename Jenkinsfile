pipeline {
    agent any
    stages {
    
        stage('Clone Repository') { 
            steps {
                git branch: 'main', url: 'https://github.com/RajeshGajengi/crud-3tier-cicd-docker-pipeline.git'
            }
        }

        stage('Docker Login') { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_cred', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}'
                }
            }
        }

        stage('Build & Push Backend Image') { 
            steps {
                sh '''
                    cd backend
                    docker build -t r25gajengi/easycrud-backend:v1 .
                    docker push r25gajengi/easycrud-backend:v1
                '''
            }
        }

        stage('Run Backend Container') {
            steps {
                withCredentials([
                    string(credentialsId: 'database_url', variable: 'SPRING_DATASOURCE_URL'),
                    string(credentialsId: 'database_user', variable: 'SPRING_DATASOURCE_USERNAME'),
                    string(credentialsId: 'database_password', variable: 'SPRING_DATASOURCE_PASSWORD')
                ]) {
                    sh '''
                    cd backend
                    docker rm -f easycrud-backend || true
                    docker run --name easycrud-backend -d \
                      -p 8081:8081 \
                      -e SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL} \
                      -e SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME} \
                      -e SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD} \
                      r25gajengi/easycrud-backend:v1
                    '''
                }
            }
        }

        stage('Build & Push Frontend Image') { 
            steps {
                withCredentials([string(credentialsId: 'backend_api', variable: 'VITE_API_URL')]) {
                    sh '''
                    cd frontend
                    echo "VITE_API_URL = ${VITE_API_URL}" > .env
                    docker build -t r25gajengi/easycrud-frontend:v1 .
                    docker push r25gajengi/easycrud-frontend:v1
                    '''
                }
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh '''
                cd frontend
                docker rm -f easycrud-frontend || true
                docker run --name easycrud-frontend -d \
                  -p 80:80 \
                  r25gajengi/easycrud-frontend:v1
                '''
            }
        }
    }
}

