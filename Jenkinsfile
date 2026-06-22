pipeline {
    agent any
    tools {
        maven 'mvn'
    }
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push michaelfawzy/myapp:${BUILD_NUMBER}'
                }
            }
        }
        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'GITHUB-CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/Ahetoo7/eyouth-argocd.git
                        cd eyouth-argocd
                        sed -i "s|image: .*myapp:.*|image: michaelfawzy/myapp:${BUILD_NUMBER}|g" apps/nginx/deployment.yaml
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git add .
                        git commit -m "update image ${BUILD_NUMBER}"
                        git push
                    '''
                }
            }
        }
    }
}