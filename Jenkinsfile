pipeline {
    agent any

    environment {
        REPO_URL = "https://github.com/Rewan0/python-app.git"
    }

    stages {
        stage('Set Environment') {
            steps {
                script {
                    def config = [
                        main: [name: 'python-hello-nti-main', port: '5000'],
                        dev : [name: 'python-hello-nti-dev',  port: '5001'],
                        stg : [name: 'python-hello-nti-stg',  port: '5002']
                    ]

                    def cfg = config[env.BRANCH_NAME]
                    if (!cfg) {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    env.APP_NAME  = cfg.name
                    env.HOST_PORT = cfg.port
                    echo "Branch: ${env.BRANCH_NAME} | App: ${env.APP_NAME} | Port: ${env.HOST_PORT}"
                }
            }
        }

        stage('Getting Repo files') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${APP_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker tag ${APP_NAME}:${BUILD_NUMBER} $DOCKER_USERNAME/${APP_NAME}:${BUILD_NUMBER}
                        docker push $DOCKER_USERNAME/${APP_NAME}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    docker rm -f ${APP_NAME} || true
                    docker run -d -p ${HOST_PORT}:5000 --name ${APP_NAME} ${APP_NAME}:${BUILD_NUMBER}
                    docker ps
                '''
            }
        }
    }
}
