pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker-hub'
        DOCKER_IMAGE = 'prathameshj08/pet-clinic'
        DOCKER_TAG = "build-${env.BUILD_NUMBER}"
        GIT_REPO = "PrathameshJ-08/spring-petclinic"
        DEPLOYMENT_FILE = "deployment/deployment.yml"
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

        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv('Sonar-server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Petclinic \
                        -Dsonar.projectName=Petclinic \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

            stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
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
                        git commit -m "Update deployment image to ${DOCKER_TAG}" || echo "No changes to commit"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_REPO}.git HEAD:main
                    '''
                }
            }
        }
    }

       post {
        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Build succeeded.</p><p><a href="${env.BUILD_URL}">View Build</a></p>""",
                mimeType: 'text/html',
                to: 'jadhavprathamesh957@gmail.com'
            )
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Build failed.</p><p><a href="${env.BUILD_URL}">View Build</a></p>""",
                mimeType: 'text/html',
                to: 'jadhavprathamesh957@gmail.com'
            )
        }
    }
}
