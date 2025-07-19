pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "prathameshj08/spring-petclinic:${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    sh '''
                    echo "nvd.api.key=$NVD_API_KEY" > dependency-check.properties

                    docker run --rm \
                    -v $(pwd):/src \
                    owasp/dependency-check \
                    --scan /src \
                    --format HTML \
                    --out /src/depcheck-report \
                    --propertyfile /src/dependency-check.properties \
                    --failOnCVSS 9.9 || true

                    rm dependency-check.properties
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
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
}
