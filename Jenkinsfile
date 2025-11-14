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

        stage('Generate Kubernetes YAMLs') {
            steps {
                script {

                    //
                    // 🔥 PURE GROOVY VARIABLE REPLACEMENT → 100% SAFE
                    //

                    // Namespace
                    def ns = readFile("k8s/namespace-template.yaml")
                    ns = ns.replace("\${APP_NAMESPACE}", APP_NAMESPACE)
                           .replace("\${APP_NAME}", APP_NAME)
                    writeFile file: "k8s/namespace.yaml", text: ns

                    // Deployment
                    def dep = readFile("k8s/deployment-template.yaml")
                    dep = dep.replace("\${APP_NAME}", APP_NAME)
                             .replace("\${APP_NAMESPACE}", APP_NAMESPACE)
                             .replace("\${IMAGE_NAME}", IMAGE_NAME)
                             .replace("\${IMAGE_TAG}", IMAGE_TAG)
                             .replace("\${APP_PORT}", APP_PORT)
                             .replace("\${REPLICA_COUNT}", REPLICA_COUNT)
                    writeFile file: "k8s/deployment.yaml", text: dep

                    // Service
                    def svc = readFile("k8s/service-template.yaml")
                    svc = svc.replace("\${APP_NAME}", APP_NAME)
                             .replace("\${APP_NAMESPACE}", APP_NAMESPACE)
                             .replace("\${APP_PORT}", APP_PORT)
                             .replace("\${NODE_PORT}", NODE_PORT)
                    writeFile file: "k8s/service.yaml", text: svc
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withEnv(["KUBECONFIG=C:\\Users\\HP\\.kube\\config"]) {

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
            echo "✅ Successfully built, generated YAML and deployed to Kubernetes!"
        }
        failure {
            echo "❌ Build failed. Check logs!"
        }
    }
}
