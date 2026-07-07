pipeline {
    agent {label 'docker-maven-trivy'}
    tools {
        maven 'maven3'
    }
    environment {
        SONAR_IP = '172.31.23.193'
        ECR_REGISTRY = '523516319028.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_REPO = "${ECR_REGISTRY}/devsecops-demo"
    }
    stages {
        stage('Trivy FS Scan'){
            steps {
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }
        stage('Build & Sonar'){
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn clean verify sonar:sonar \
  -Dsonar.projectKey=devsecops-demo \
  -Dsonar.host.url="http://${SONAR_IP}:9000" \
  -Dsonar.token="${SONAR_TOKEN}" \
  -Dsonar.qualitygate.wait=true'// some block
                }
            }
        }
        stage('ECR Login'){
            steps {
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $ECR_REGISTRY'
            }
        }
        stage('Build Image') {
            steps {
                sh 'export DOCKER_BUILDKIT=0 && docker build --platform linux/amd64 -t "$IMAGE_REPO:$BUILD_NUMBER" -t "$IMAGE_REPO:latest" .'
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL "$IMAGE_REPO:$BUILD_NUMBER"'
            }
        }
    }
}
