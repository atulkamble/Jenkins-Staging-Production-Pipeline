pipeline {
    agent any

    environment {
        IMAGE_NAME = "myapp"
        REGISTRY = "123456789.dkr.ecr.us-east-1.amazonaws.com"
        STAGING_SERVER = "ubuntu@staging-server-ip"
        PRODUCTION_SERVER = "ubuntu@production-server-ip"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repository.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package' // For Java
                // sh 'npm install && npm run build' // For Node.js
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test' // Run unit tests
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:latest .
                    docker tag ${IMAGE_NAME}:latest ${REGISTRY}/${IMAGE_NAME}:latest
                    docker push ${REGISTRY}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sh """
                    ssh ${STAGING_SERVER} 'docker pull ${REGISTRY}/${IMAGE_NAME}:latest'
                    ssh ${STAGING_SERVER} 'docker stop myapp-container || true'
                    ssh ${STAGING_SERVER} 'docker run -d --rm --name myapp-container -p 8080:8080 ${REGISTRY}/${IMAGE_NAME}:latest'
                    """
                }
            }
        }

        stage('Approval for Production Deployment') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    sh """
                    ssh ${PRODUCTION_SERVER} 'docker pull ${REGISTRY}/${IMAGE_NAME}:latest'
                    ssh ${PRODUCTION_SERVER} 'docker stop myapp-container || true'
                    ssh ${PRODUCTION_SERVER} 'docker run -d --rm --name myapp-container -p 8080:8080 ${REGISTRY}/${IMAGE_NAME}:latest'
                    """
                }
            }
        }
    }
}
