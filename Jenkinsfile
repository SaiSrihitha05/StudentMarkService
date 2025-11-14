pipeline {
    agent any

    environment {
        APP_NAME       = "studentmarkservice"
        APP_NAMESPACE  = "${APP_NAME}-ns"
        IMAGE_NAME     = "${APP_NAME}-image"
        IMAGE_TAG      = "${BUILD_NUMBER}${BUILD_NUMBER}"
        APP_PORT       = "8100"
        NODE_PORT      = "30081"
        REPLICA_COUNT  = "2"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/SaiSrihitha05/StudentMarkService.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f Dockerfile .
                """
            }
        }

        stage('K8s Deployment') {
            steps {
                script {
                        withEnv(["KUBECONFIG=C:\\Users\\HP\\.kube\\config"]) {

                        // ✔ FIXED: No backslashes, fully Groovy-safe
                        powershell '''
(Get-Content k8s/namespace-template.yaml) `
    -replace '\${APP_NAMESPACE}', '${APP_NAMESPACE}' `
    -replace '\${APP_NAME}', '${APP_NAME}' |
    Set-Content k8s/namespace.yaml

(Get-Content k8s/deployment-template.yaml) `
    -replace '\${APP_NAME}', '${APP_NAME}' `
    -replace '\${APP_NAMESPACE}', '${APP_NAMESPACE}' `
    -replace '\${IMAGE_NAME}', '${IMAGE_NAME}' `
    -replace '\${IMAGE_TAG}', '${IMAGE_TAG}' `
    -replace '\${APP_PORT}', '${APP_PORT}' `
    -replace '\${REPLICA_COUNT}', '${REPLICA_COUNT}' |
    Set-Content k8s/deployment.yaml

(Get-Content k8s/service-template.yaml) `
    -replace '\${APP_NAME}', '${APP_NAME}' `
    -replace '\${APP_NAMESPACE}', '${APP_NAMESPACE}' `
    -replace '\${APP_PORT}', '${APP_PORT}' `
    -replace '\${NODE_PORT}', '${NODE_PORT}' |
    Set-Content k8s/service.yaml
'''

                        bat "kubectl apply -f k8s/namespace.yaml --validate=false"
                        bat "kubectl apply -f k8s/deployment.yaml --validate=false"
                        bat "kubectl apply -f k8s/service.yaml --validate=false"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Checkout, Build, Dockerize & Deploy completed successfully!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}
