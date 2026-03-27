pipeline {
    agent any

    tools {
        maven 'Maven-3'
        jdk 'JDK21'
    }

    stages {

        stage('Build') {
            steps {
                sh './mvn clean install'
            }
        }

    }
}
