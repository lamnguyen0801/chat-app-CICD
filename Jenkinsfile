pipeline {
    agent {
        label 'AGENT-01'
    }
    tools {
        nodejs 'nodeJS-18.20.8'
    }
    options {
        skipDefaultCheckout true
    }
    environment {
        SONAR_TOKEN = credentials('token-chatapp-sona')
        NAME_FRONTEND = 'chat-fe'
        NAME_BACKEND = 'chat-be'
        DOCKER_HUB = 'nguyenlam0801'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credential')
    }
    stages {
        stage('Clone Source') {
            steps {
                git credentialsId: 'jenkins-gitlab-user-1',
                    url: 'http://lamnm.gitlab/chatapp/chatapp.git',
                    branch: 'main'
            }
        }
        stage('Build FE') {
            steps {
                sh """
                    cd public
                    yarn install --frozen-lockfile
                    CI=false
                    yarn build
                    cd ..
                """
            }
        }
        stage('Build BE') {
            steps {
                sh """
                    cd server
                    yarn install --frozen-lockfile
                    CI=false
                    cd ..
                """
            }
        }
        stage('Test with sonarqube in FE') {
            steps {
                dir('public') {
                    withSonarQubeEnv('SonarQube_Server') {
                        sh '''
                            docker run --rm -e SONAR_HOST_URL=${SONAR_HOST_URL} \
                            -e SONAR_SCANNER_OPTS='-Dsonar.projectKey=ChatApp_FE' \
                            -e SONAR_TOKEN=${SONAR_TOKEN} \
                            -v '.:/usr/src/fe' \
                            sonarsource/sonar-scanner-cli
                        '''
                    }  
                }
            }
        }
        stage('Test with sonarqube in BE') {
            steps {
                dir('server') {
                    withSonarQubeEnv('SonarQube_Server') {
                        sh '''
                            docker run --rm -e SONAR_HOST_URL=${SONAR_HOST_URL} \
                            -e SONAR_SCANNER_OPTS='-Dsonar.projectKey=ChatApp_BE' \
                            -e SONAR_TOKEN=${SONAR_TOKEN} \
                            -v '.:/usr/src/fe' \
                            sonarsource/sonar-scanner-cli
                        '''
                    }
                }
            }
        }
        stage('Build and push image') {
            steps {
                script {
                    def commit = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    env.GIT_COMMIT = commit
                    env.DOCKER_TAG = commit.take(7)
                    env.IMAGE_TAG = env.DOCKER_TAG
                    sh ''' 
                        IMAGE_TAG=${IMAGE_TAG}
                        NAME_BACKEND=${NAME_BACKEND}
                        NAME_FRONTEND=${NAME_FRONTEND}
                        docker compose build --parallel
                        docker tag ${NAME_BACKEND}:${DOCKER_TAG} ${DOCKER_HUB}/${NAME_BACKEND}:${DOCKER_TAG}
                        docker tag ${NAME_FRONTEND}:${DOCKER_TAG} ${DOCKER_HUB}/${NAME_FRONTEND}:${DOCKER_TAG}
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_HUB}/${NAME_BACKEND}:${DOCKER_TAG}
                        docker push ${DOCKER_HUB}/${NAME_FRONTEND}:${DOCKER_TAG}
                        docker rmi ${DOCKER_HUB}/${NAME_BACKEND}:${DOCKER_TAG} || echo "Image not removed (still used or not found)"
                        docker rmi ${DOCKER_HUB}/${NAME_FRONTEND}:${DOCKER_TAG} || echo "Image not removed (still used or not found)"
                        docker rmi ${NAME_BACKEND}:${DOCKER_TAG} || echo "Image not removed (still used or not found)"
                        docker rmi ${NAME_FRONTEND}:${DOCKER_TAG} || echo "Image not removed (still used or not found)"
                    '''
                }
            }
        } 
        stage('Deploy'){
            steps {
                script {
                    def deploying = "#!/bin/bash\n" +
                        "docker rm -f ${NAME_BACKEND} ${NAME_FRONTEND}\n" +
                        "docker pull ${DOCKER_HUB}/${NAME_BACKEND}:${DOCKER_TAG}\n" +
                        "docker pull ${DOCKER_HUB}/${NAME_FRONTEND}:${DOCKER_TAG}\n" +
                        "docker run --name=${NAME_BACKEND} --env-file /home/test/.env --dns=8.8.8.8 -dp 5000:5000 ${DOCKER_HUB}/${NAME_BACKEND}:${DOCKER_TAG}\n" +
                        "docker run --name=${NAME_FRONTEND} -dp 3000:80 ${DOCKER_HUB}/${NAME_FRONTEND}:${DOCKER_TAG}"

                    withCredentials([file(credentialsId: 'chatapp-env', variable: 'ENV_FILE')]) {
                        sshagent(credentials: ['jenkins-ssh-key']) {
                            sh """
                                scp -i jenkins-ssh-key -o StrictHostKeyChecking=no $ENV_FILE test@192.168.72.105:/home/test/.env
                                ssh -o StrictHostKeyChecking=no -i jenkins-ssh-key test@192.168.72.105 "echo \\\"${deploying}\\\" > deploy.sh && chmod +x deploy.sh && ./deploy.sh"
                            """
                        }
                    }
                }
            }
        }
    }
}

// CI=false => bỏ CI default của react (warning trên CI thành error)
// Cần có chính sách clean up storage registry vì dù image có cùng tag thì khi push lên thì digest cũ vẫn còn
// Stage build rồi vẫn còn bước build code trong Dockerfile vì
// 1. Build code trước giúp kiểm tra lỗi sớm (early feedback)
// 2. Tester (hoặc QA pipeline) có thể test code trước
// 3. Tiết kiệm storage (và cả thời gian)

// biến SONAR_TOKEN trong environment {}	
// Trong sh "..." 1 dòng	Có thể dùng ${SONAR_TOKEN}
// Trong sh ''' ... ''' nhiều dòng	Phải dùng ${env.SONAR_TOKEN} hoặc """ ... """ nhiều dòng cũng dùng được ${SONAR_TOKEN}

// Luôn dùng ' (single quotes) trong câu lệnh sh(...) hoặc bat(...) khi thao tác với credentials để tránh nội suy Groovy làm lộ thông tin
// https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#string-interpolation
