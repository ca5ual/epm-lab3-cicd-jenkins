pipeline {
    agent any

    tools {
        nodejs "node" // use what I configured in tools
    }
    
    environment {
        DOCKER_IMAGE = "ca5ual/lab3"
    }

    stages {
        
        stage ('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage ('Run tests') {
            steps {
                sh 'npm test'
            }
        }

        stage ('Set ENV') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.ENV = 'nodemain'
                        env.JOB = 'Deploy_to_main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.ENV = 'nodedev'
                        env.JOB = 'Deploy_to_dev'
                    }
                }
            }
        }

        stage ('Build docker image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${ENV}-v1.0 ."
                }
            }
        }

        stage('Push to docker') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                        docker push ${DOCKER_IMAGE}:${ENV}-v1.0
                        """
                    }
                }
            }
        }

        stage ('Trigger deploy') {
            steps {
                script {
                    build job: env.JOB
                }
            }
        }
    }
}