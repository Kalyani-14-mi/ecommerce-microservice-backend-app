pipeline {
    agent any

    tools {
        maven 'Maven-3'
        jdk 'Java'
    }

    stages {

        stage('Build') {
            steps {
                sh './mvn clean install'
            }
        }

    }
}
