pipeline {
    agent any

    

    environment {
        SONARQUBE = 'SonarQubeServer'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Replace with actual Jenkins credential ID
    }

    stages {

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/shreya-ghub/voting-app-ci.git', branch: 'main'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh "sonar-scanner -Dsonar.projectKey=voting-app -Dsonar.sources=example-voting-app/"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    cd example-voting-app
                    docker build -t shreyaramki/vote:latest ./vote
                    docker build -t shreyaramki/result:latest ./result
                    docker build -t shreyaramki/worker:latest ./worker
                    '''
                }
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                script {
                    sh '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push shreyaramki/vote:latest
                    docker push shreyaramki/result:latest
                    docker push shreyaramki/worker:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                cd example-voting-app
                kubectl apply -f k8s-specifications/
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sh '''
                cd ansible
                ansible-playbook -i inventory.ini playbook.yml
                '''
            }
        }
    }
}

