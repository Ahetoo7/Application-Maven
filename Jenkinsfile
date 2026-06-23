pipeline {
    agent {
        kubernetes {
            cloud 'k8s-cloud'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins

  containers:

    - name: maven
      image: maven:3.9-eclipse-temurin-17
      command: ["cat"]
      tty: true

    - name: docker
      image: docker:24-cli
      command: ["cat"]
      tty: true
      volumeMounts:
        - name: docker-sock
          mountPath: /var/run/docker.sock

    - name: git
      image: alpine/git:2.45.2
      command: ["cat"]
      tty: true

  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
'''
        }
    }

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
                        rm -rf eyouth-argocd
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/Ahetoo7/eyouth-argocd.git
                        cd eyouth-argocd
                        sed -i "s|image: .*|image: michaelfawzy/myapp:${BUILD_NUMBER}|g" apps/nginx3/deployment.yaml
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