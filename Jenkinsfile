pipeline {
    agent any
    tools {
        maven 'MAVEN-3'
        jdk 'JDK21'
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
    }
}
