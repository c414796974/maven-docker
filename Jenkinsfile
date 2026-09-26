pipeline {
    agent any

    environment {
        HARBOR_ADDR    = "10.15.125.141"
        HARBOR_PROJECT = "cicd-nginx"
        IMAGE_NAME     = "nginx-demo"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
        HARBOR_CREDS   = credentials('harbor-auth')
        APP_HOST       = "10.15.125.143"
    }

    stages {

        stage('拉取代码') {
            steps {
                checkout scm
            }
        }

        stage('构建镜像') {
            steps {
                sh """
                    docker build \
                      -t ${HARBOR_ADDR}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('推送到Harbor') {
            steps {
                sh """
                    echo '${HARBOR_CREDS_PSW}' | docker login ${HARBOR_ADDR} \
                      -u '${HARBOR_CREDS_USR}' \
                      --password-stdin

                    docker push \
                      ${HARBOR_ADDR}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}

                    docker logout ${HARBOR_ADDR}
                """
            }
        }

        stage('部署到应用服务器') {
            steps {
                sh """
                    sshpass -p '24W_WrM7?SAwSL@Q' ssh \
                      -o StrictHostKeyChecking=no \
                      -o UserKnownHostsFile=/dev/null \
                      root@${APP_HOST} <<EOF

echo '${HARBOR_CREDS_PSW}' | docker login ${HARBOR_ADDR} \
  -u '${HARBOR_CREDS_USR}' \
  --password-stdin

docker stop nginx-cicd-demo 2>/dev/null || true
docker rm nginx-cicd-demo 2>/dev/null || true

docker pull ${HARBOR_ADDR}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}

docker run -d \
  --name nginx-cicd-demo \
  -p 80:80 \
  ${HARBOR_ADDR}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}

docker logout ${HARBOR_ADDR}

EOF
                """
            }
        }
    }

    post {
        always {
            echo "流水线执行完成，镜像版本号：${IMAGE_TAG}"
        }
    }
}