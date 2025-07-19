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
        DEPLOYMENT_FILE = "deployment.yml"
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                git branch: 'main', credentialsId: 'github-account', url: "https://github.com/${GIT_REPO}.git"
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

        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config --global user.email "jadhavprathamesh957@gmail.com"
                        git config --global user.name "Prathamesh"

                        sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|" ${DEPLOYMENT_FILE}

                        git add ${DEPLOYMENT_FILE}
                        git commit -m "Update deployment image to ${DOCKER_TAG}" || echo "No changes to commit"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_REPO}.git HEAD:main
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
            echo "✅ Build succeeded!"
        }
        failure {
            echo "❌ Build failed. Check the logs for details."
        }
    }
}
