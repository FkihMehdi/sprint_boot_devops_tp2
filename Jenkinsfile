pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('2')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
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
                    def imageName = "amir145/devops_tp3"
                    sh "docker build -t ${imageName}:0 ."
                    sh "docker tag ${imageName}:0 ${imageName}:latest"
                }
            }
        }

        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push') {
            steps {
                script {
                    def imageName = "amir145/devops_tp3"
                    sh "docker push ${imageName}:latest"
                }
            }
        }

        stage('Install kubectl') {
            steps {
                sh '''
                    curl -LO "https://dl.k8s.io/release/v1.23.0/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mkdir -p $HOME/bin
                    mv kubectl $HOME/bin/
                    export PATH=$HOME/bin:$PATH
                '''
            }
        }

        stage('Install Minikube') {
            steps {
                sh '''
                    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
                    chmod +x minikube
                    mkdir -p $HOME/bin
                    mv minikube $HOME/bin/
                    export PATH=$HOME/bin:$PATH
                '''
            }
        }

        stage('Déployer sur Kubernetes') {
            steps {
                script {
                    sh 'minikube start'
                    sh 'minikube kubectl apply -f deployment.yaml'
                    sh 'minikube kubectl apply -f service.yaml'
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
