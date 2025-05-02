pipeline{

    agent any


    tools {
        maven 'Maven'
    }

	environment {
		DOCKERHUB_CREDENTIALS=credentials('1')
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

        stage('Déployer sur Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
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