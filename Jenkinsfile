pipeline {
    agent any

    environment {
        REGISTRY = "crpi-29ns4nxq5xxuk3v4.cn-hangzhou.personal.cr.aliyuncs.com" // 阿里云容器镜像服务地址
        IMAGE_NAME = "spring-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}" 
        DOCKER_CREDENTIALS = "docker-creds"   // Jenkins中配置的docker仓库凭据ID
        KUBECONFIG_CREDENTIALS = "kubeconfig" // Jenkins中配置的kubeconfig凭据ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'http://114.55.230.186/root/spring-javademo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", DOCKER_CREDENTIALS) {
                        def app = docker.build("${REGISTRY}/myteam/${IMAGE_NAME}:${IMAGE_TAG}")
                        app.push()
                        app.push("latest") // 可选，更新latest标签
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS, variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${REGISTRY}/myteam/${IMAGE_NAME}:${IMAGE_TAG} -n default
                        kubectl rollout status deployment/${IMAGE_NAME} -n default
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ 部署成功: ${REGISTRY}/myteam/${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ 构建或部署失败，请检查日志"
        }
    }
}
