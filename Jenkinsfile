pipeline {
    agent any

    environment {
        ImageRegistry = 'ferdinandjrdocker'
        EC2_IP = '18.139.84.36'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
        Dimage = 'barks_meows_paradise1'
        DOCKERHUB_CREDENTIALS = credentials('ferdinandjrdocker')
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh 'docker build -t ferdinandjrdocker/barks_meows_paradise1:1.0 .'
                }
            }
        }


        stage("Login") {
            steps {
                script {
                    echo "Login"
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    }
                }
            }

        stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                        sh "docker push ${ImageRegistry}/${Dimage}:1.0 "
                    }
                }
            }
        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    sshagent(['ec2']) {
                        // Upload files once to reduce redundant SCP commands
                        sh """
                        scp -o StrictHostKeyChecking=no ubuntu@${EC2_IP}:/home/ubuntu
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} down"
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} up -d"
                        """
                    }
                }
            }
        }
        
        }

    
    }
