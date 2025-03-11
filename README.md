# Jenkins-Staging-Production-Pipeline
### **Jenkins Pipeline Development: Staging and Production Deployment**

A Jenkins pipeline for deploying applications to **staging** and **production** environments follows a structured workflow that includes building, testing, and deploying. Below is a breakdown of the **multi-stage Jenkins pipeline** for seamless deployment.

---

## **Pipeline Workflow**
1. **Source Code Checkout**: Clone the repository from GitHub/GitLab/Bitbucket.
2. **Build the Application**: Use Maven, Gradle, or npm for building the code.
3. **Run Tests**: Execute unit and integration tests.
4. **Containerization (Optional)**: Build a Docker image if using containers.
5. **Deploy to Staging**: Deploy to a **staging** environment for testing.
6. **Approval Step**: Requires manual approval for production deployment.
7. **Deploy to Production**: If approved, deploy to the production environment.

---

## **Jenkinsfile for Staging & Production Pipeline**
Here’s a sample **Declarative Jenkins Pipeline** with **staging and production** deployment:

```groovy
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
```

---

## **Explanation**
- **Checkout Code**: Pulls the latest code from the Git repository.
- **Build Application**: Uses **Maven** (or npm for JavaScript) to build the app.
- **Run Tests**: Executes unit tests.
- **Docker Build & Push**: Builds a Docker image and pushes it to a container registry (AWS ECR in this case).
- **Deploy to Staging**: Pulls and runs the latest Docker image in the **staging server**.
- **Approval Step**: Requires manual confirmation before proceeding to production.
- **Deploy to Production**: Pulls and runs the latest image on the **production server**.

---

## **Key Features**
✔ **Automated Build & Test**  
✔ **Staging Deployment for Verification**  
✔ **Manual Approval Before Production**  
✔ **Zero-Downtime Docker Deployments**  
✔ **Supports AWS ECR, Docker, and SSH-based Deployment**  

Would you like modifications to support **Kubernetes (EKS) deployments** or **AWS CodeDeploy** instead of SSH-based Docker deployments? 🚀
