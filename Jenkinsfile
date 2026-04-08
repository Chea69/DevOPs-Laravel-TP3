pipeline {
    agent { label 'laravel' }

    environment {
        TELEGRAM_TOKEN   = credentials('telegram-token')
        TELEGRAM_CHAT_ID = credentials('telegram-chat-id')
        SERVER_HOST      = credentials('server-host')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Chea69/DevOPs-Laravel-TP3.git'
            }
        }

        stage('Build Laravel') {
            steps {
                sh '''
                    composer install
                    cp .env.example .env || true
                    php artisan key:generate
                    npm install
                    npm run build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['server-ssh']) {
                    sh """
                        ansible-playbook -i inventory.ini deploy.yml \
                          --extra-vars "server_host=${SERVER_HOST}"
                    """
                }
            }
        }
    }

    post {
        success {
            sh """
                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
                -d "chat_id=${TELEGRAM_CHAT_ID}" \
                -d "text=✅ Build success: ${JOB_NAME} #${BUILD_NUMBER}"
            """
        }
        failure {
            sh """
                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
                -d "chat_id=${TELEGRAM_CHAT_ID}" \
                -d "text=❌ Build failed: ${JOB_NAME} #${BUILD_NUMBER}"
            """
        }
    }
}