pipeline {
    agent {
        kubernetes {
            cloud 'k8s-cloud'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:

    - name: maven
      image: maven:3.9-eclipse-temurin-17
      command: ["sleep"]
      args: ["99d"]

    - name: docker
      image: docker:24-dind
      securityContext:
        privileged: true
      env:
        - name: DOCKER_TLS_CERTDIR
          value: ""

    - name: git
      image: alpine/git:2.45.2
      command: ["sleep"]
      args: ["99d"]
'''
        }
    }

    stages {

        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Docker Build') {
            steps {
                container('docker') {
                    sh '''
dockerd-entrypoint.sh &
sleep 5

docker version
docker build -t michaelfawzy/myapp:${BUILD_NUMBER} .
'''
                }
            }
        }

        stage('Docker Push') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh '''
echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
docker push michaelfawzy/myapp:${BUILD_NUMBER}
'''
                    }
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                container('git') {
                    withCredentials([usernamePassword(
                        credentialsId: 'GITHUB-CRED',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {

                        sh '''
rm -rf eyouth-argocd
git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/Ahetoo7/eyouth-argocd.git
cd eyouth-argocd

sed -i "s|image: .*|image: michaelfawzy/myapp:${BUILD_NUMBER}|g" apps/nginx3/deployment.yaml

git config user.email "jenkins@ci.local"
git config user.name "Jenkins CI"

git add .
git commit -m "update image ${BUILD_NUMBER}" || true
git push
'''
                    }
                }
            }
        }
    }
}