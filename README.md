# CI/CD Pipeline for 3-Tier App Deployment with Docker and Jenkins

This project demonstrates the deployment of a 3-tier application architecture using Docker, Jenkins, Java (Maven) for the backend, Node.js (npm) for the frontend, and AWS RDS (MariaDB) for the database. The application follows a CI/CD pipeline to automate the build and deployment process, ensuring efficient and reliable updates to the production environment.


## Overview

This project consists of:
- `Backend`: A REST API built using Java and Spring Boot (Maven).
- `Frontend`: A React-based web application built using Node.js and npm.
- `Database`: A MariaDB instance hosted on AWS RDS.

The CI/CD pipeline automates the process of building, testing, and deploying both backend and frontend applications in Docker containers.

## Project Structure
```
📁 backend/        → Java Spring Boot backend  
📁 frontend/       → Node.js frontend (React + Vite)  
📄 Jenkinsfile     → Jenkins pipeline definition  
📄 README.md       → Project documentation  
```

## Tech Stack

- **Frontend**: React, Vite, Node.js, NPM  
- **Backend**: Java, Spring Boot, Maven  
- **Database**: MariaDB (AWS RDS)  
- **CI/CD**: Jenkins  
- **Containerization**: Docker  
- **Cloud**: AWS    

## Prerequisites

Ensure you have the following tools installed before starting the setup:
 - `Jenkins`: For continuous integration and continuous deployment (CI/CD).
 - `Docker`: To build and run the backend and frontend applications as containers.
 - `Terraform`: For provisioning and managing infrastructure (optional but recommended for cloud deployments).
 - `AWS CLI`: For interacting with AWS services.
 - `Kubernetes`: (Optional) For container orchestration (if you're using Kubernetes).
 - `MariaDB Client`: For connecting to the MariaDB database hosted on AWS RDS.
 - `Java (JDK)`: To build the backend service using Maven.
 - `Node.js`: To build the frontend service with npm.


## Database Setup (MariaDB on AWS RDS)
Database Setup (MariaDB on AWS RDS)

This project uses a MariaDB database hosted on AWS RDS.

### Step 1: Create the RDS instance

If you haven’t already, create a MariaDB-compatible RDS instance via the AWS Console or using Terraform.
- Engine: MariaDB
- Port: 3306
- Public access: Yes (or configure a proper VPC + security group if running privately)
- Database username: e.g., `admin`
- Password: e.g., `yourpassword`

### Step 2: Connect to the RDS instance
Use the MariaDB client to connect to your RDS instance:
```bash
mysql -h <RDS_ENDPOINT> -u admin -p
```
You’ll be prompted to enter the password.

### Step 3: Create the database
Once connected:
```sql
CREATE DATABASE student_db;
```
This will create the database used by the backend Spring Boot application.

⚠️ Make sure the name matches the value in the `SPRING_DATASOURCE_URL` (e.g., `jdbc:mariadb://<RDS_ENDPOINT>:3306/student_db`).

### Step 4: Grant access (optional)
If using a different DB user than admin, ensure it has privileges on the database:
```sql
GRANT ALL PRIVILEGES ON student_db.* TO 'your_user'@'%' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
```

**Notes:**

- Make sure the RDS security group allows inbound access on port 3306 from the IP or EC2/Jenkins host.
- Ensure the RDS endpoint and credentials are added in Jenkins credentials:
    - `database_url`
    - `database_user`
    - `database_password`

## Set Up Jenkins and Docker

### 1. Add Jenkins User to Docker Group

To allow Jenkins to interact with Docker, add the Jenkins user to the Docker group:

```bash
sudo usermod -aG docker jenkins

```
Then restart Jenkins:

```bash
sudo systemctl restart jenkins

```

### 2. Configure Jenkins Credentials
In Jenkins, add the following credentials under Manage Jenkins → Manage Credentials:

 - **docker_cred**: Docker Hub username and password.

 - **database_url**: `jdbc:mariadb://<RDS_Endpoint>:3306/student_db`

 - **database_user**: Database username for the MariaDB instance.

 - **database_password**: Database password for the MariaDB instance.

 - **backend_api**: Backend API URL, e.g., `http://<backend_IP>:8081/api`.



## Backend Setup (Java + Maven)
### 1. Clone the Repository:
```bash
git clone
```

### 2. Update application.properties: 
The backend is a Java-based Spring Boot application that is built using Maven.
In the `application.properties` file, replace the database connection properties with environment variables:

```
server.port=8081
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

```

## Frontend Setup (Node.js + NPM)
1. **Frontend Framework**: The frontend is built using React and Node.js.

2. **Environment Configuration**: The frontend communicates with the backend API and requires setting the `VITE_API_URL` environment variable.

## Environment Variables

| Variable                    | Used In   | Description                              |
|----------------------------|-----------|------------------------------------------|
| SPRING_DATASOURCE_URL      | Backend   | JDBC URL for MariaDB                     |
| SPRING_DATASOURCE_USERNAME | Backend   | DB username                              |
| SPRING_DATASOURCE_PASSWORD | Backend   | DB password                              |
| VITE_API_URL               | Frontend  | URL to access backend API                |



## CI/CD Pipeline

The project uses a Jenkins pipeline for continuous integration and deployment. Below is an explanation of the stages in the Jenkins pipeline:

### Jenkins Pipeline Script
```groovy
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


```

## Pipeline Stages:
1. **Clone Repository**:

- Clones the GitHub repository containing the source code.

2. **Docker Login**:

- Logs into Docker Hub using the stored Docker credentials.

3. **Build & Push Backend Image**:

- Runs the Maven build for the backend and creates a Docker image, which is then pushed to Docker Hub.

4. **Run Backend Container**:

- Runs the backend container on port 8081 and passes database credentials as environment variables.

5. **Build & Push Frontend Image**:

- Installs npm dependencies and creates the frontend Docker image. The image is pushed to Docker Hub.

6. **Run Frontend Container**:

- Runs the frontend container on port 80.


## Deployment Process
1.**Docker Build and Push**

- The Backend and Frontend applications are containerized using Docker.
- The Docker images are built and pushed to Docker Hub.

2. **Environment Variables**

- The backend service connects to the AWS RDS MariaDB database using environment variables for the database URL, username, and password.
- The frontend service is configured to use the backend API URL dynamically via the environment variable VITE_API_URL.

3. **Running Containers**
- The backend is exposed on port 8081, and the frontend is exposed on port 80.


## Run Locally (Manual Steps)
### Backend
```bash
cd backend
# With locally 
mvn clean install
cd target
java -jar student-registration-backend-0.0.1-SNAPSHOT.jar

# with docker
docker build -t backend:local .
docker run -p 8081:8081 backend:local

```
### Frontend

```bash
cd frontend
echo "VITE_API_URL=http://Backend_IP:8081/api" > .env

# With locally (make sure you have installed apache2)
npm install
npm run build
cp -rf dist/* /var/www/html

# With docker
docker build -t frontend:local .
docker run -p 80:80 frontend:local

```

## Conclusion

This project demonstrates the use of a Jenkins pipeline for continuous integration and deployment of a 3-tier application, with the backend built using Java (Maven) and the frontend built using Node.js (npm). By leveraging Docker and AWS RDS, the system is both scalable and efficient. The CI/CD pipeline automates the process of building, pushing, and running containers, ensuring seamless deployments and updates.