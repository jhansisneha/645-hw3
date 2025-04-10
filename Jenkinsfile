pipeline {
    agent any

    environment {
        IMAGE = 'jhansisneha/swe645-hw3:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        docker build -t $IMAGE .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kube_config', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                        mkdir -p ~/.kube
                        cp \$KUBECONFIG_FILE ~/.kube/config
                        chmod 600 ~/.kube/config
                        kubectl apply -f k8s/survey-deployment.yaml
                        kubectl apply -f k8s/survey-service.yaml
                    """
                }
            }
        }
    }
}