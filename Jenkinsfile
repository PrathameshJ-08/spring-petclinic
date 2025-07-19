pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = 'prathameshj08/pet-clinic'
        DOCKER_TAG = "build-${env.BUILD_NUMBER}"
        GIT_REPO = "PrathameshJ-08/spring-petclinic"
        DEPLOYMENT_FILE = "deployment/deployment.yml"
    }

    stages {
        stage('Checkout') {
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
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker logout
                    """
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
                        git commit -m "Update deployment to ${DOCKER_TAG}" || echo "No changes"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_REPO}.git HEAD:main
                    '''
                }
            }
        }
    }

    post {
        always {
            mail to: 'jadhavprathamesh957@gmail.com',
                 subject: "Build ${env.BUILD_NUMBER} - ${currentBuild.result}",
                 body: "Check the build at ${env.BUILD_URL}"
        }
    }
}
