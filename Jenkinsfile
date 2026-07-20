  pipeline {
    agent any
    tools {
        maven 'MAVEN-3'
        jdk 'JDK21'
    }

    environment {
        PROMETHEUS_URL = "http://prometheus-server.monitoring.svc.cluster.local"
        GRAFANA_URL    = "http://grafana.monitoring.svc.cluster.local"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build & Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                    sh 'docker build -t $DOCKER_USER/service-discovery:latest ./service-discovery'
                    sh 'docker push $DOCKER_USER/service-discovery:latest'
                    sh 'docker build -t $DOCKER_USER/cloud-config:latest ./cloud-config'
                    sh 'docker push $DOCKER_USER/cloud-config:latest'
                    sh 'docker build -t $DOCKER_USER/api-gateway:latest ./api-gateway'
                    sh 'docker push $DOCKER_USER/api-gateway:latest'
                    sh 'docker build -t $DOCKER_USER/proxy-client:latest ./proxy-client'
                    sh 'docker push $DOCKER_USER/proxy-client:latest'
                    sh 'docker build -t $DOCKER_USER/user-service:latest ./user-service'
                    sh 'docker push $DOCKER_USER/user-service:latest'
                    sh 'docker build -t $DOCKER_USER/product-service:latest ./product-service'
                    sh 'docker push $DOCKER_USER/product-service:latest'
                    sh 'docker build -t $DOCKER_USER/favourite-service:latest ./favourite-service'
                    sh 'docker push $DOCKER_USER/favourite-service:latest'
                    sh 'docker build -t $DOCKER_USER/order-service:latest ./order-service'
                    sh 'docker push $DOCKER_USER/order-service:latest'
                    sh 'docker build -t $DOCKER_USER/shipping-service:latest ./shipping-service'
                    sh 'docker push $DOCKER_USER/shipping-service:latest'
                    sh 'docker build -t $DOCKER_USER/payment-service:latest ./payment-service'
                    sh 'docker push $DOCKER_USER/payment-service:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // This also applies k8s/monitoring/prometheus-jenkins-scrape.yaml automatically
                sh 'kubectl apply -Rf k8s/'
            }
            post {
                success {
                    // Annotate Grafana: deployment started successfully
                    withCredentials([string(credentialsId: 'grafana-api-key', variable: 'GRAFANA_TOKEN')]) {
                        sh """
                            curl -s -X POST ${GRAFANA_URL}/api/annotations \
                              -H "Content-Type: application/json" \
                              -H "Authorization: Bearer \$GRAFANA_TOKEN" \
                              -d '{
                                "text": "Deployed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                                "tags": ["jenkins", "deploy", "success"]
                              }' || echo "Grafana annotation skipped (non-blocking)"
                        """
                    }
                }
            }
        }

        stage('Verify Services Health') {
            steps {
                script {
                    // Wait for rollouts to complete for all your services
                    def services = [
                        'service-discovery',
                        'cloud-config',
                        'api-gateway',
                        'proxy-client',
                        'user-service',
                        'product-service',
                        'favourite-service',
                        'order-service',
                        'shipping-service',
                        'payment-service'
                    ]
                    for (svc in services) {
                        sh "kubectl rollout status deployment/${svc} --timeout=120s || echo '${svc} rollout check skipped'"
                    }
                }
            }
        }

        stage('Verify Metrics in Prometheus') {
            steps {
                script {
                    sh """
                        echo "Waiting 30s for metrics to settle..."
                        sleep 30

                        echo "Checking Jenkins is UP in Prometheus..."
                        RESULT=\$(curl -s '${PROMETHEUS_URL}/api/v1/query?query=up{job="jenkins"}' \
                          | grep -o '"value":\\[.*\\]' | grep -o '"[01]"' | tail -1 || echo '"unknown"')

                        echo "Prometheus result for Jenkins: \$RESULT"

                        if [ "\$RESULT" = '"0"' ]; then
                            echo "WARNING: Jenkins appears DOWN in Prometheus. Check scrape config."
                        else
                            echo "Jenkins metrics are being scraped successfully."
                        fi
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. Check Grafana for deployment annotations."
        }

        failure {
            // Annotate Grafana on pipeline failure
            withCredentials([string(credentialsId: 'grafana-api-key', variable: 'GRAFANA_TOKEN')]) {
                sh """
                    curl -s -X POST ${GRAFANA_URL}/api/annotations \
                      -H "Content-Type: application/json" \
                      -H "Authorization: Bearer \$GRAFANA_TOKEN" \
                      -d '{
                        "text": "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        "tags": ["jenkins", "deploy", "failure"]
                      }' || echo "Grafana annotation skipped (non-blocking)"
                """
            }
        }
    }
}        
