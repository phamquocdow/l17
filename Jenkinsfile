pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myapp"
        DOCKER_TAG   = "${env.BUILD_NUMBER}"
    }

    stages {

        // ============================================================
        // STAGE DÙNG CHUNG CHO MỌI NHÁNH (tuỳ chọn)
        // ============================================================
        stage('Checkout') {
            steps {
                echo "Đang checkout branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        // ============================================================
        // 1. NHÁNH feature/* -> Unit Test + Lint
        // ============================================================
        stage('Unit Test') {
            when {
                branch 'feature/*'
            }
            steps {
                echo "==> [feature/*] Chạy Unit Test..."
                sh 'echo "npm install"'
                sh 'echo "npm test"'
                // sh 'npm install'
                // sh 'npm test'
            }
        }

        stage('Linting') {
            when {
                branch 'feature/*'
            }
            steps {
                echo "==> [feature/*] Kiểm tra lỗi cú pháp (Lint)..."
                sh 'echo "npm run lint"'
                // sh 'npm run lint'
            }
        }

        // ============================================================
        // 2. NHÁNH develop -> Build Docker image + Deploy Staging
        // ============================================================
        stage('Build Docker Image (Develop)') {
            when {
                branch 'develop'
            }
            steps {
                echo "==> [develop] Build Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh 'echo "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."'
                // sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo "==> [develop] Deploy lên môi trường Staging..."
                sh 'echo "kubectl apply -f k8s/staging.yaml"'
                // sh 'kubectl --context=staging apply -f k8s/staging.yaml'
            }
        }

        // ============================================================
        // 3. NHÁNH main -> Yêu cầu xác nhận thủ công rồi Deploy Production
        // ============================================================
        stage('Build Docker Image (Main)') {
            when {
                branch 'main'
            }
            steps {
                echo "==> [main] Build Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh 'echo "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."'
                // sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Manual Approval') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Pipeline sẽ dừng tại đây chờ admin bấm Approve
                    // timeout để tránh pipeline treo vô hạn nếu không ai duyệt
                    timeout(time: 24, unit: 'HOURS') {
                        input message: "Xác nhận Deploy lên PRODUCTION?",
                              ok: "Approve",
                              submitter: "admin"
                    }
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo "==> [main] Đã được duyệt. Deploy lên Production..."
                sh 'echo "kubectl apply -f k8s/production.yaml"'
                // sh 'kubectl --context=production apply -f k8s/production.yaml'
            }
        }
    }

    post {
        success {
            echo "Pipeline hoàn thành thành công cho branch ${env.BRANCH_NAME}."
        }
        failure {
            echo "Pipeline thất bại trên branch ${env.BRANCH_NAME}."
        }
        aborted {
            echo "Pipeline bị huỷ (có thể do không ai Approve deploy Production)."
        }
    }
}
