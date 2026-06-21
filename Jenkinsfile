pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t michaelfawzy/myapp:${BUILD_NUMBER} .'
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push michaelfawzy/myapp:${BUILD_NUMBER}'
            }
        }
    }
}