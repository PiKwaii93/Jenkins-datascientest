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
                sh '''
                rm -rf .venv
                python3 -m venv .venv
                .venv/bin/python -m pip install --upgrade pip
                .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Testing') {
            steps {
                sh '.venv/bin/python -m unittest'
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
            input {
                message "Proceed to push to main"
                ok "Yes"
            }

            steps {
                echo 'User acceptance approved'
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
                        sh "echo 'Merging done'"
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
