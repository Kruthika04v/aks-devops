pipeline {
    agent any

    environment {
        REGISTRY = "krut0402.azurecr.io"
        IMAGE = "flask-app"
        IMAGE_TAG = "latest"
        RESOURCE_GROUP = "Cnapp-RG"
        CLUSTER_NAME = "devops-aks"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Kruthika04v/aks-devops.git'
            }
        }

        stage('Login to Azure Container Registry') {
            steps {
                sh '''
                az acr login --name krut0402
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $REGISTRY/$IMAGE:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image to ACR') {
            steps {
                sh '''
                docker push $REGISTRY/$IMAGE:$IMAGE_TAG
                '''
            }
        }

        stage('Get AKS Credentials') {
            steps {
                sh '''
                az aks get-credentials \
                    --resource-group $RESOURCE_GROUP \
                    --name $CLUSTER_NAME \
                    --overwrite-existing
                '''
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Get External IP') {
            steps {
                sh '''
                echo "Waiting for LoadBalancer IP..."
                sleep 60

                kubectl get svc flask-service

                echo "----------------------------------"
                echo "EXTERNAL IP (clean output):"
                kubectl get svc flask-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
                echo ""
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed! Check logs."
        }
    }
}
