pipeline {
    agent any

    // Environment variables used across stages
    environment {
        IMAGE_NAME = "yourdockerhubusername/helloworld-java"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"   // You must configure this in Jenkins
        KUBECONFIG = "/etc/rancher/k3s/k3s.yaml"          // Path to your Kubernetes config (k3s by default)
    }

    stages {

        // ----------------------------
        // 1. Clone the GitHub repo
        // ----------------------------
        stage('Checkout Source Code') {
            steps {
                echo "Cloning project from GitHub..."
                git url: 'https://github.com/yourusername/helloworld-springboot.git', branch: 'main'
            }
        }

        // ----------------------------
        // 2. Build Docker image
        // ----------------------------
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image from Dockerfile..."
                script {
                    // Build the Docker image using the current directory's Dockerfile
                    dockerImage = docker.build("${IMAGE_NAME}:latest")
                }
            }
        }

        // ----------------------------
        // 3. Push Docker image to Docker Hub
        // ----------------------------
        stage('Push Image to Docker Hub') {
            steps {
                echo "Logging in and pushing image to Docker Hub..."
                script {
                    // Use the Docker Hub credentials stored in Jenkins
                    docker.withRegistry('', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("latest")
                    }
                }
            }
        }

        // ----------------------------
        // 4. Deploy app to Kubernetes
        // ----------------------------
        stage('Deploy to Kubernetes') {
            steps {
                echo "Creating Kubernetes deployment YAML and applying to cluster..."
                script {
                    // Create the Kubernetes manifest file dynamically
                    writeFile file: 'k8s-deployment.yaml', text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: ${IMAGE_NAME}:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: helloworld-service
spec:
  type: LoadBalancer
  selector:
    app: helloworld
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
"""

                    // Apply the manifest using kubectl
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }

    // ----------------------------
    // 5. Post-pipeline notification
    // ----------------------------
    post {
        success {
            echo "✅ App deployed successfully to Kubernetes!"
        }
        failure {
            echo "❌ Deployment failed. Check the logs above."
        }
    }
}
