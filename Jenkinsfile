pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "prathameshj08/spring-petclinic:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PrathameshJ-08/spring-petclinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    sh '''
                        docker run --rm \
                        -e NVD_API_KEY=$NVD_API_KEY \
                        -v $(pwd):/src \
                        owasp/dependency-check \
                        --scan /src \
                        --format HTML \
                        --out /src/depcheck-report \
                        --failOnCVSS 9.9 || true
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'depcheck-report/*.html', allowEmptyArchive: true
        }

        success {
            mail to: 'jadhavprathamesh957@gmail.com',
                 subject: "SUCCESS: PetClinic Build #${BUILD_NUMBER}",
                 body: "✅ Build succeeded and Docker image pushed: $DOCKER_IMAGE"
        }

        failure {
            mail to: 'jadhavprathamesh957@gmail.com',
                 subject: "FAILURE: PetClinic Build #${BUILD_NUMBER}",
                 body: "❌ Build failed. Please check Jenkins console output."
        }
    }
}
