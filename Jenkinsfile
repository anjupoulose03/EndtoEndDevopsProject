pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'sonar'    // Name configured in Jenkins → Manage Jenkins → Configure System
        AWS_REGION = "us-east-1"
        ECR_REPO = "334712111310.dkr.ecr.us-west-1.amazonaws.com/springboot-app"
        IMAGE_TAG = "v1"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/anjupoulose03/EndtoEndDevopsProject.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                        mvn clean package sonar:sonar \
                        -Dsonar.projectKey=endtoend-app \
                        -Dsonar.projectName="EndtoEnd App"
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t endtoend-app:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REPO}
                """
            }
        }

        stage('Tag & Push Image to ECR') {
            steps {
                sh """
                    docker tag endtoend-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name my-eks-cluster
                    kubectl apply -f k8s-deployment.yaml
                    kubectl apply -f k8s-service.yaml
                """
            }
        }

    }
}

