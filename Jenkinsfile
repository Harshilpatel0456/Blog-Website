pipeline {
    // Run this pipeline on any available agent
    agent any

    environment {
        // ID of credentials stored in Jenkins Credentials Manager
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        
        // Docker Hub namespace / username
        DOCKER_HUB_USER = 'harshilpatel0456'
        
        // Target image repositories
        BACKEND_IMAGE = "${DOCKER_HUB_USER}/blog-website-backend"
        FRONTEND_IMAGE = "${DOCKER_HUB_USER}/my-frontend"
        
        // Version tag for deployment
        IMAGE_TAG = "latest"
    }

    stages {
        // ==========================================
        // Stage 1: Checkout Source Code
        // ==========================================
        stage('Checkout') {
            steps {
                echo "Fetching repository source code..."
                checkout scm
            }
        }

        // ==========================================
        // Stage 2: Lint Client Application
        // ==========================================
        stage('Client Lint') {
            tools {
                nodejs 'NodeJS'
            }
            steps {
                echo "Running linter on the React application..."
                dir('client') {
                    // Install packages and run linter (non-blocking for demo purposes if errors exist)
                    sh 'npm install'
                    sh 'npm run lint -- --fix || true'
                }
            }
        }

        // ==========================================
        // Stage 3: Build Docker Images
        // ==========================================
        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building Docker Images using Docker Compose..."
                    sh "docker compose build"
                }
            }
        }

        // ==========================================
        // Stage 4: Publish Images to Registry
        // ==========================================
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Authenticating and pushing images to Docker Hub registry..."
                    // withCredentials binds the password variable dynamically, preventing it from being leaked in console outputs
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker push ${BACKEND_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                    }
                }
            }
        }

        // ==========================================
        // Stage 5: Deploy Services Locally
        // ==========================================
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying application containers locally using docker compose..."
                    // Shut down running services and re-create them with the latest image versions
                    sh "docker compose down"
                    sh "docker compose up -d"
                }
            }
        }
    }

    post {
        // ==========================================
        // Post-Execution Cleanup and Notifications
        // ==========================================
        always {
            echo "Performing cleanup..."
            // Remove unused and dangling Docker images to optimize storage
            sh "docker image prune -f"
        }
        success {
            echo "CI/CD Pipeline finished successfully!"
        }
        failure {
            echo "CI/CD Pipeline failed! Check logs above."
        }
    }
}
