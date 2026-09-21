pipeline {
    agent any

    // ─────────────────────────────────────────────
    // Poll SCM: kiểm tra thay đổi trên GitHub mỗi 5 phút
    // ─────────────────────────────────────────────
    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        // ── Thông tin Git repo ──
        GIT_REPO_URL  = 'https://github.com/hunglxt37/test-linux'
        GIT_BRANCH    = 'main'
        GIT_CRED_ID   = 'github-token'

        // ── Thư mục trên VPS nơi chứa source code ──
        DEPLOY_DIR    = '/opt/smartgrocery'

        // ── Thông tin SSH vào VPS ──
        // Tạo Credential loại "SSH Username with private key" trong Jenkins với ID = 'vps-ssh-key'
        SSH_CRED_ID   = 'vps-ssh-key'
        VPS_HOST      = '192.168.139.128'
        VPS_USER      = 'thanhhung'

        // ── Prefix để tag image (không cần Docker Hub, build & dùng local) ──
        DOCKERHUB_USERNAME = 'smartgrocery'

        // ── Tên project cho docker compose ──
        COMPOSE_PROJECT = 'smartgrocery'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        // ─────────────────────────────────────────
        // Stage 1: Kiểm tra kết nối SSH tới VPS
        // ─────────────────────────────────────────
        stage('1. Verify VPS Connection') {
            steps {
                echo '=== Kiểm tra kết nối SSH tới VPS ==='
                withCredentials([sshUserPrivateKey(credentialsId: SSH_CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${VPS_USER}@${VPS_HOST} \
                            'echo "SSH OK – host: \$(hostname) – uptime: \$(uptime -p)"'
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // Stage 2: Tạo thư mục & pull code mới nhất trên VPS
        // ─────────────────────────────────────────
        stage('2. Prepare & Pull Source on VPS') {
            steps {
                echo '=== Tạo thư mục repo và pull code mới nhất trên VPS ==='
                withCredentials([sshUserPrivateKey(credentialsId: SSH_CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${VPS_USER}@${VPS_HOST} '
                            set -e
                            mkdir -p ${DEPLOY_DIR}

                            if [ ! -d "${DEPLOY_DIR}/.git" ]; then
                                echo ">>> Repo chua co – clone lan dau..."
                                git clone --branch ${GIT_BRANCH} ${GIT_REPO_URL} ${DEPLOY_DIR}
                            else
                                echo ">>> Repo da co – pull ban moi nhat..."
                                cd ${DEPLOY_DIR}
                                git fetch --all
                                git checkout ${GIT_BRANCH}
                                git reset --hard origin/${GIT_BRANCH}
                            fi

                            cd ${DEPLOY_DIR}
                            echo "=== Commit hien tai ==="
                            git log --oneline -3
                        '
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // Stage 3: Build Docker Images trên VPS
        // ─────────────────────────────────────────
        stage('3. Build Docker Images on VPS') {
            steps {
                echo '=== Build Docker images trực tiếp trên VPS ==='
                withCredentials([sshUserPrivateKey(credentialsId: SSH_CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${VPS_USER}@${VPS_HOST} '
                            set -e
                            cd ${DEPLOY_DIR}

                            echo ">>> Build backend image..."
                            docker build -t ${DOCKERHUB_USERNAME}/smartgrocery-backend:latest ./backend

                            echo ">>> Build frontend image..."
                            docker build -t ${DOCKERHUB_USERNAME}/smartgrocery-frontend:latest ./frontend

                            echo "=== Images vua build ==="
                            docker images | grep smartgrocery
                        '
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // Stage 4: Deploy với Docker Compose
        // ─────────────────────────────────────────
        stage('4. Deploy with Docker Compose') {
            steps {
                echo '=== Deploy ứng dụng bằng docker-compose ==='
                withCredentials([sshUserPrivateKey(credentialsId: SSH_CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${VPS_USER}@${VPS_HOST} '
                            set -e
                            cd ${DEPLOY_DIR}

                            if [ ! -f .env ]; then
                                echo ">>> Tao .env mac dinh..."
                                printf "DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME}\\nDB_USERNAME=postgres\\nDB_PASSWORD=123456\\n" > .env
                            fi

                            echo ">>> Dung stack cu..."
                            docker compose -p ${COMPOSE_PROJECT} down --remove-orphans || true

                            echo ">>> Khoi dong stack moi..."
                            docker compose -p ${COMPOSE_PROJECT} up -d

                            echo "Deploy hoan tat!"
                        '
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // Stage 5: Health Check
        // ─────────────────────────────────────────
        stage('5. Health Check') {
            steps {
                echo '=== Kiểm tra trạng thái các container sau deploy ==='
                withCredentials([sshUserPrivateKey(credentialsId: SSH_CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${VPS_USER}@${VPS_HOST} '
                            echo "--- Trang thai docker compose ---"
                            docker compose -p ${COMPOSE_PROJECT} ps

                            echo "--- Cho 10 giay cho service khoi dong ---"
                            sleep 10

                            echo "--- Kiem tra frontend (port 80) ---"
                            curl -sf http://localhost:80 > /dev/null \\
                                && echo "Frontend OK" \\
                                || echo "Frontend chua san sang"

                            echo "--- Kiểm tra backend (port 8080) ---"
                            curl -sf http://localhost:8080/actuator/health > /dev/null \\
                                && echo "Backend OK" \\
                                || echo "Backend chua san sang (co the can them thoi gian)"
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "[SUCCESS] Pipeline hoan tat! Ung dung da duoc trien khai len VPS: ${VPS_HOST}"
        }
        failure {
            echo '[FAILED] Pipeline that bai! Kiem tra Console Output de xem chi tiet.'
        }
        always {
            echo "=== Pipeline ket thuc: ${currentBuild.currentResult} – Build #${BUILD_NUMBER} ==="
        }
    }
}