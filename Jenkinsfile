pipeline {
    agent any

    tools {
        maven 'Maven-3'
        jdk 'Java21'
    }

    stages {

        stage('Build') {
            steps {
                sh './mvn clean install'
            }
        }

    }
}
