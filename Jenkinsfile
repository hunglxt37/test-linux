pipeline {
    agent any

    // ── Biến môi trường ──────────────────────────────────────────
    environment {
        DOCKERHUB_USERNAME = credentials('DOCKERHUB_USERNAME') // Secret text
        DOCKERHUB_TOKEN    = credentials('DOCKERHUB_TOKEN')    // Secret text
        IMAGE_BACKEND      = "${DOCKERHUB_USERNAME}/smartgrocery-backend"
        IMAGE_FRONTEND     = "${DOCKERHUB_USERNAME}/smartgrocery-frontend"

        // VPS nơi Jenkins đang chạy
        VPS_USER = 'thanhhung'
        VPS_HOST = '192.168.139.128'
        // VPS_SSH_KEY: Jenkins Credential loại "SSH Username with private key"
        // ID credential: VPS_SSH_KEY
    }

    triggers {
        githubPush()
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {

        // ─────────────────────────────────────────────────────────
        // STAGE 1: Checkout
        // ─────────────────────────────────────────────────────────
        stage('Checkout') {
            steps {
                echo '📥 Cloning source code...'
                checkout scm
            }
        }

        // ─────────────────────────────────────────────────────────
        // STAGE 2: Test Backend (Spring Boot + H2 in-memory)
        // ─────────────────────────────────────────────────────────
        stage('Backend Test') {
            steps {
                echo '🧪 Running backend unit tests...'
                dir('backend') {
                    sh 'chmod +x gradlew'
                    sh './gradlew test --no-daemon'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'backend/build/test-results/test/*.xml'
                }
            }
        }

        // ─────────────────────────────────────────────────────────
        // STAGE 3: Build Frontend (Vue 3 + Vite)
        // ─────────────────────────────────────────────────────────
        stage('Frontend Build') {
            steps {
                echo '⚡ Building Vue 3 frontend...'
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        // ─────────────────────────────────────────────────────────
        // STAGE 4: Build & Push Docker Images lên Docker Hub
        // (chỉ chạy khi push lên nhánh main)
        // ─────────────────────────────────────────────────────────
        stage('Docker Build & Push') {
            when { branch 'main' }
            steps {
                echo '🐳 Building and pushing Docker images...'

                // Đăng nhập Docker Hub
                sh 'echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin'

                // Build & push Backend
                sh """
                    docker build -t ${IMAGE_BACKEND}:latest \
                                 -t ${IMAGE_BACKEND}:${GIT_COMMIT} \
                                 ./backend
                    docker push ${IMAGE_BACKEND}:latest
                    docker push ${IMAGE_BACKEND}:${GIT_COMMIT}
                """

                // Build & push Frontend
                sh """
                    docker build -t ${IMAGE_FRONTEND}:latest \
                                 -t ${IMAGE_FRONTEND}:${GIT_COMMIT} \
                                 ./frontend
                    docker push ${IMAGE_FRONTEND}:latest
                    docker push ${IMAGE_FRONTEND}:${GIT_COMMIT}
                """

                sh 'docker image prune -f'
            }
        }

        // ─────────────────────────────────────────────────────────
        // STAGE 5: Deploy trên VPS
        // Jenkins đang chạy TRONG container trên VPS này,
        // dùng SSH vào host machine để chạy docker compose.
        // ─────────────────────────────────────────────────────────
        stage('Deploy to VPS') {
            when { branch 'main' }
            steps {
                echo '🚀 Deploying on VPS (thanhhung@192.168.139.128)...'

                // Dùng sshagent plugin để quản lý SSH key an toàn
                // Credential ID 'VPS_SSH_KEY' được cấu hình trong Jenkins
                sshagent(credentials: ['VPS_SSH_KEY']) {

                    // Copy docker-compose.yml lên VPS host
                    sh """
                        scp -o StrictHostKeyChecking=no \
                            docker-compose.yml \
                            ${VPS_USER}@${VPS_HOST}:~/smartgrocery/
                    """

                    // SSH vào host VPS và chạy docker compose
                    sh """
                        ssh -o StrictHostKeyChecking=no \
                            ${VPS_USER}@${VPS_HOST} '
                                set -e
                                cd ~/smartgrocery

                                export DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME}

                                echo "⬇️  Pulling latest images..."
                                docker compose pull

                                echo "🔄 Restarting containers..."
                                docker compose up -d --remove-orphans

                                echo "🧹 Cleaning old images..."
                                docker image prune -f

                                echo "✅ Deploy thành công!"
                                docker compose ps
                            '
                    """
                }
            }
        }
    }

    // ── Thông báo sau khi pipeline kết thúc ──────────────────────
    post {
        success {
            echo '✅ Pipeline hoàn tất thành công!'
        }
        failure {
            echo '❌ Pipeline thất bại! Kiểm tra log bên trên.'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
