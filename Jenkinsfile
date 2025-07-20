pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub'
        DOCKER_IMAGE = 'prathameshj08/pet-clinic'
        DOCKER_TAG = "build-${env.BUILD_NUMBER}"
        GIT_REPO = "PrathameshJ-08/spring-petclinic"
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                git branch: 'main', credentialsId: 'github-account', url: "https://github.com/${GIT_REPO}.git"
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Remote Deployment to Kube-VM') {
            steps {
                sshagent(['kube-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ditiss@192.168.150.20 \"
                        sed -i 's|image: prathameshj08/pet-clinic:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|' ~/Desktop/Project/deployment.yml &&
                        kubectl apply -f ~/Desktop/Project/deployment.yml &&
                        kubectl rollout restart deployment petclinic-deployment
                    \"
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Build finished: ${currentBuild.result}"
        }
        success {
            echo "✅ Build and deployment succeeded!"
        }
        failure {
            echo "❌ Build or deployment failed. Check Jenkins logs."
        }
    }
}
