pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "prathameshj08/spring-petclinic:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/PrathameshJ-08/spring-petclinic.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Run Dependency Check') {
            steps {
                sh '''
                docker run --rm \
                --volume $(pwd):/src \
                owasp/dependency-check \
                --scan /src \
                --format HTML \
                --out /src/depcheck-report
                '''
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
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
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
        success {
            mail to: 'jadhavprathamesh957@gmail.com',
                 subject: "SUCCESS: PetClinic Build #${BUILD_NUMBER}",
                 body: "Build Success!"
        }
        failure {
            mail to: 'jadhavprathamesh957@gmail.com',
                 subject: "FAILURE: PetClinic Build #${BUILD_NUMBER}",
                 body: "Build Failed!"
        }
    }
}
