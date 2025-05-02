pipeline {
    agent any
    
    environment {
        // Make sure this matches exactly with the ID you created in Jenkins
        DOCKER_HUB_CREDS = credentials('1')
        // Update with your actual Docker Hub username and app name
        DOCKER_IMAGE = "amir145/devops_tp3"
    }

    tools {
        maven 'Maven'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Application') {
            steps {
                // Use sh for Windows commands instead of sh
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:0 ."
                    sh "docker tag ${DOCKER_IMAGE}:0 ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
                    sh "docker push ${DOCKER_IMAGE}:0"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                    sh "docker logout"
                }
            }
        }

         stage('Déployer sur Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                cleanWs()
                sh "docker rmi ${DOCKER_IMAGE}:0 || exit 0"
                sh "docker rmi ${DOCKER_IMAGE}:latest || exit 0"
            }
        }
    }
}