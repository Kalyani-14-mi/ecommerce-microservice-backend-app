pipeline {
    agent any

    tools {
        maven 'Maven'
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
