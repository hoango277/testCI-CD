pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-secret
      secret:
        secretName: dockerhub-secret
"""
        }
    }
    environment {
        DOCKER_IMAGE = "xuanhoa2772004/ci-cd-learn:${BUILD_NUMBER}"
        DEPLOYMENT_FILE = "app/deployment.yaml"
        GIT_CI_REPO = "https://github.com/hoango277/testCI-CD.git"
        GIT_CD_REPO = "https://github.com/hoango277/CD-VDT.git"
        GIT_BRANCH = "main"
        GIT_CRED = "95173ae0-f1e9-433b-a956-0777fccc7f0c" // GitHub credential ID
    }
    stages {
        stage('Checkout CI Repo') {
            steps {
                dir('ci') {
                    git branch: "${GIT_BRANCH}", credentialsId: "${GIT_CRED}", url: "${GIT_CI_REPO}"
                }
            }
        }
        stage('Build & Push Image') {
            steps {
                dir('ci') {
                    container('kaniko') {
                        sh """
                        /kaniko/executor --dockerfile=Dockerfile --context=${WORKSPACE}/ci --destination=${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }
        stage('Checkout CD Repo') {
            steps {
                dir('cd') {
                    git branch: "${GIT_BRANCH}", credentialsId: "${GIT_CRED}", url: "${GIT_CD_REPO}"
                }
            }
        }
        stage('Update deployment.yaml') {
            steps {
                dir('cd') {
                    // Sửa tag image trong deployment.yaml
                    sh """
                    sed -i 's|image: xuanhoa2772004/ci-cd-learn:.*|image: ${DOCKER_IMAGE}|' ${DEPLOYMENT_FILE}
                    """
                }
            }
        }
        stage('Commit & Push to CD Repo') {
            steps {
                dir('cd') {
                    withCredentials([usernamePassword(credentialsId: "${GIT_CRED}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                        git config user.email "ci-bot@yourdomain.com"
                        git config user.name "ci-bot"
                        git add ${DEPLOYMENT_FILE}
                        git commit -m "Update image tag to ${DOCKER_IMAGE} [ci skip]" || echo "No changes to commit"
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/hoango277/CD-VDT.git HEAD:${GIT_BRANCH}
                        """
                    }
                }
            }
        }
    }
}