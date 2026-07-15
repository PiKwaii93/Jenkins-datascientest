pipeline {
    agent any

    environment {
        DOCKER_ID = "pikawaii93"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {
        stage('Building') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Testing') {
            steps {
                sh 'python3 -m unittest'
            }
        }

        stage('Deploying') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('User Acceptance') {
            steps {
                input {
                    message "Proceed to push to main"
                    ok "Yes"
                }
            }
        }

        stage('Pushing and Merging') {
            parallel {
                stage('Pushing Image') {
                    environment {
                        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
                    }

                    steps {
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                    }
                }

                stage('Merging') {
                    steps {
                        echo 'Merging done'
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
