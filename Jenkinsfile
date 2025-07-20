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
        DEPLOYMENT_FILE = "deployment.yml"
        KUBE_VM = "ditiss@192.168.150.20"
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                git branch: 'main', credentialsId: 'github-account', url: 'https://github.com/PrathameshJ-08/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
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
                sshagent (credentials: ['kube-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${KUBE_VM} '
                            sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|" ~/Desktop/Project/deployment.yml &&
                            kubectl apply -f ~/Desktop/Project/deployment.yml
                        '
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Build finished: ${currentBuild.result}"
        }
        success {
            echo "✅ Build succeeded and deployed to Kube-VM!"
        }
        failure {
            echo "❌ Build or Deployment failed. Check Jenkins logs."
        }
    }
}
