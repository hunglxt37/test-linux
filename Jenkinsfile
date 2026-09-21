pipeline {
    agent any

    // ── Biến môi trường ──────────────────────────────────────────
    environment {
        DOCKERHUB_USERNAME  = credentials('DOCKERHUB_USERNAME')   // Jenkins Credential (Secret text)
        DOCKERHUB_TOKEN     = credentials('DOCKERHUB_TOKEN')      // Jenkins Credential (Secret text)
        IMAGE_BACKEND       = "${DOCKERHUB_USERNAME}/smartgrocery-backend"
        IMAGE_FRONTEND      = "${DOCKERHUB_USERNAME}/smartgrocery-frontend"

        VPS_HOST            = credentials('VPS_HOST')             // Jenkins Credential (Secret text)
        VPS_USER            = credentials('VPS_USER')             // Jenkins Credential (Secret text)
        VPS_PORT            = credentials('VPS_PORT')             // Jenkins Credential (Secret text)
        VPS_SSH_KEY         = credentials('VPS_SSH_KEY')          // Jenkins Credential (SSH Username with private key)
    }

    // ── Trigger tự động khi push lên nhánh main ──────────────────
    triggers {
        githubPush()
    }

    // ── Options ──────────────────────────────────────────────────
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {

        // STAGE 1: Checkout
        stage('Checkout') {
            steps {
                echo '📥 Cloning source code...'
                checkout scm
            }
        }

        // STAGE 2: Test Backend (Spring Boot + H2)
        stage('Backend Test') {
            steps {
                echo '🧪 Running backend unit tests...'
                dir('backend') {
                    sh 'chmod +x gradlew'
                    sh './gradlew test --no-daemon'
                }
            }
            post {
                failure {
                    echo '❌ Backend tests failed!'
                    junit 'backend/build/test-results/test/*.xml'
                }
                success {
                    junit 'backend/build/test-results/test/*.xml'
                }
            }
        }

        // STAGE 3: Build Frontend (Vue 3 + Vite)
        stage('Frontend Build') {
            steps {
                echo '⚡ Building Vue 3 frontend...'
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        // STAGE 4: Build & Push Docker Images
        // (chỉ chạy khi build branch main)
        stage('Docker Build & Push') {
            when {
                branch 'main'
            }
            steps {
                echo '🐳 Building and pushing Docker images...'

                // Đăng nhập Docker Hub
                sh 'echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin'

                // Build & push Backend image
                sh """
                    docker build -t ${IMAGE_BACKEND}:latest \\
                                 -t ${IMAGE_BACKEND}:${GIT_COMMIT} \\
                                 ./backend
                    docker push ${IMAGE_BACKEND}:latest
                    docker push ${IMAGE_BACKEND}:${GIT_COMMIT}
                """

                // Build & push Frontend image
                sh """
                    docker build -t ${IMAGE_FRONTEND}:latest \\
                                 -t ${IMAGE_FRONTEND}:${GIT_COMMIT} \\
                                 ./frontend
                    docker push ${IMAGE_FRONTEND}:latest
                    docker push ${IMAGE_FRONTEND}:${GIT_COMMIT}
                """

                // Dọn dẹp image local sau khi push
                sh 'docker image prune -f'
            }
        }

        // STAGE 5: Deploy lên VPS qua SSH
        // (chỉ chạy khi build branch main)
        stage('Deploy to VPS') {
            when {
                branch 'main'
            }
            steps {
                echo '🚀 Deploying to VPS...'

                // Copy docker-compose.yml lên VPS
                sh """
                    scp -i ${VPS_SSH_KEY} \\
                        -o StrictHostKeyChecking=no \\
                        -P ${VPS_PORT} \\
                        docker-compose.yml \\
                        ${VPS_USER}@${VPS_HOST}:~/smartgrocery/
                """

                // SSH vào VPS và deploy
                sh """
                    ssh -i ${VPS_SSH_KEY} \\
                        -o StrictHostKeyChecking=no \\
                        -p ${VPS_PORT} \\
                        ${VPS_USER}@${VPS_HOST} '
                            cd ~/smartgrocery
                            export DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME}
                            docker compose pull
                            docker compose up -d --remove-orphans
                            docker image prune -f
                            echo "✅ Deploy thành công!"
                        '
                """
            }
        }
    }

    // ── Post actions (sau khi pipeline kết thúc) ─────────────────
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
