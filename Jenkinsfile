pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        SONARQUBE_SERVER = 'SonarQubeServer'
        DOCKER_IMAGE_VOTE = "shreyaramki/vote"
        DOCKER_IMAGE_RESULT = "shreyaramki/result"
        DOCKER_IMAGE_WORKER = "shreyaramki/worker"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/shreya-ghub/voting-app-ci.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'sonar-scanner -Dsonar.projectKey=voting-app -Dsonar.sources=example-voting-app/'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE_VOTE ./example-voting-app/vote
                docker build -t $DOCKER_IMAGE_RESULT ./example-voting-app/result
                docker build -t $DOCKER_IMAGE_WORKER ./example-voting-app/worker
                '''
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $DOCKER_IMAGE_VOTE
                    docker push $DOCKER_IMAGE_RESULT
                    docker push $DOCKER_IMAGE_WORKER
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s-deployment/vote.yaml
                kubectl apply -f k8s-deployment/result.yaml
                kubectl apply -f k8s-deployment/worker.yaml
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sh 'ansible-playbook -i inventory deploy-voting-app.yaml'
            }
        }
    }
}

