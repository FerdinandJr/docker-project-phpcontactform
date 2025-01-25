pipeline {
    agent any

    environment {
        ImageRegistry = 'ferdinandjrdocker'
        EC2_IP = '18.139.84.36'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
        Dimage = 'barks_meows_paradise1'
        DOCKERHUB_CREDENTIALS = credentials('ferdinandjrdocker')
        DockerImageTag = "${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER}"
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh 'docker build -t ${DockerImageTag} .'
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
                        sh "docker push ${DockerImageTag} "
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
                        scp -o StrictHostKeyChecking=no ${DockerComposeFile} ubuntu@${EC2_IP}:/home/ubuntu
                        export DC_IMAGE_NAME=${DockerImageTag} && \
                        echo ${DC_IMAGE_NAME} && \
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} down"
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} up -d"
                        """
                    }
                }
            }
        }
        
        }

    
    }
