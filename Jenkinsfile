pipeline {
    agent any

    environment {
        TAG         = "latest"
        REGISTRY    = "${env.REGISTRY_URL}"

        IMAGE_WEB   = "${env.IMAGE_WEB}:${TAG}"
        IMAGE_DB    = "${env.IMAGE_DB}:${TAG}"
        IMAGE_NGINX = "${env.IMAGE_NGINX}:${TAG}"

        COMPOSE_FILE = "${env.COMPOSE_FILE}"

    }

    stages {

        stage('Checkout') {
            steps {
                sh 'echo "Checking out repository..."'
                checkout scm
                slackSend channel: '#ci-devopstonytec', message: "Checkout iniciado", tokenCredentialId: 'slack-token'
            }
        }

        stage('Build Images') {
            steps {
                script {
                    // Alterado para apontar para o novo ID 'dockerhub-token'
                    withDockerRegistry([credentialsId: 'dockerhub-token', url: 'https://index.docker.io/v1/']) {
                
                         // Seus comandos de docker build e docker push entram aqui
                
                    }
                }
            }
        }

       stage('Security Scan (Trivy)') {
            steps {
                slackSend channel: '#ci-devopstonytec', message: "Rodando Trivy Scan...", tokenCredentialId: 'slack-token'

                sh """
                    trivy image ${IMAGE_WEB}   --severity HIGH,CRITICAL --exit-code 0 --format json --output trivy-web.json
                    trivy image ${IMAGE_DB}    --severity HIGH,CRITICAL --exit-code 0 --format json --output trivy-db.json
                    trivy image ${IMAGE_NGINX} --severity HIGH,CRITICAL --exit-code 0 --format json --output trivy-nginx.json
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-*.json', fingerprint: true
                    slackSend channel: '#ci-devopstonytec', message: "Trivy scan concluído. Relatórios disponíveis nos artefatos.", tokenCredentialId: 'slack-token'
                }
            }
        }

        stage('Test in Containers') {
            steps {
                slackSend channel: '#ci-devopstonytec', message: "Subindo containers para testes...", tokenCredentialId: 'slack-token'

                sh """
                    docker rm -f web1 web2 web3 db nginx || true
                    docker compose -f ${COMPOSE_FILE} up -d

                    sleep 5
                    curl -I http://localhost || exit 1

                    docker compose -f ${COMPOSE_FILE} down
                """
            }
        }

        stage('Deploy to Production') {
            steps {
                slackSend channel: '#ci-devopstonytec', message: "Deploy em produção iniciado...", tokenCredentialId: 'slack-token'

                sh """
                    docker compose pull
                    docker compose up -d
                """
            }
        }
    }

    post {
        success {
            slackSend channel: '#ci-devopstonytec', message: "Pipeline finalizado com sucesso! Build #${BUILD_NUMBER}", tokenCredentialId: 'slack-token'
        }
        failure {
            slackSend channel: '#ci-devopstonytec', message: "Pipeline falhou! Verifique o console. Build #${BUILD_NUMBER}", tokenCredentialId: 'slack-token'
        }
    }
}


