pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
        readOnly: true
  volumes:
    - name: kaniko-secret
      secret:
        secretName: dockerhub-secret
        items:
          - key: .dockerconfigjson
            path: config.json
"""
        }
    }
    environment {
        DOCKER_IMAGE = "xuanhoa2772004/ci-cd-learn:${BUILD_NUMBER}"
        DEPLOYMENT_FILE = "app/deployment.yaml"
        GIT_CI_REPO = "https://github.com/hoango277/testCI-CD.git"
        GIT_CD_REPO = "https://github.com/hoango277/CD-VDT.git"
        GIT_BRANCH = "main"
        GIT_CRED = "95173ae8-fde9-433b-a956-0777f0cc4f0c"
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
                        echo '==> CHECK KANIKO DOCKER CONFIG:'
                        ls -la /kaniko/.docker/
                        cat /kaniko/.docker/config.json || echo "No config.json found"
                        echo '==> START BUILD WITH DEBUG:'
                        /kaniko/executor --dockerfile=Dockerfile --context=. --destination=${DOCKER_IMAGE} --verbosity=debug
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
