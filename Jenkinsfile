pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub'
        DOCKER_IMAGE = 'prathameshj08/pet-clinic'
        GIT_REPO = "PrathameshJ-08/spring-petclinic"
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                git branch: 'main', credentialsId: 'github-account', url: "https://github.com/${GIT_REPO}.git"
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${DOCKER_IMAGE}:latest .
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent (credentials: ['kube-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ditiss@kube-VM "
                        cd /home/ditiss/Desktop/Project &&
                        kubectl apply -f deployment.yml &&
                        kubectl rollout restart deployment petclinic-deployment
                    "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Build completed with status: ${currentBuild.result}"
        }
        success {
            echo "✅ Build and deployment succeeded!"
        }
        failure {
            echo "❌ Build or deployment failed!"
        }
    }
}
