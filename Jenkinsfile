pipeline {
    agent any

    environment {
        CD_REPO = 'https://github.com/echuwok12/Argotest.git'
        DEPLOY_PATH = 'deploy/tripapp'         
        IMAGE = "tripsplitter/tripapp"
        NAMESPACE = "tripapp"
    }

    stages {
        stage('Checkout CD Repo') {
            steps {
                script {
                    sh 'rm -rf cd-repo || true'
                    sh "git clone ${CD_REPO} cd-repo"
                }
            }
        }

        stage('Update Image Tag') {
            steps {
                script {
                    // Assume the CI pipeline passes IMAGE_TAG as a parameter or env var
                    if (!env.IMAGE_TAG) {
                        error "IMAGE_TAG not provided! CI pipeline must pass the image tag."
                    }

                    dir("cd-repo/${DEPLOY_PATH}") {
                        sh '''
                        echo "Updating image tag to ${IMAGE}:${IMAGE_TAG}"
                        yq -i ".spec.template.spec.containers[0].image = \\"${IMAGE}:${IMAGE_TAG}\\"" deployment.yaml
                        '''
                    }
                }
            }
        }

        stage('Git Commit and Push Changes') {
            steps {
                dir('cd-repo') {
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh '''
                        git config --global user.name "jenkins-bot"
                        git config --global user.email "jenkins@local"
                        git add .
                        git commit -m "CD: update image to ${IMAGE}:${IMAGE_TAG}" || echo "No changes to commit"
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/echuwok12/Argotest.git HEAD:main
                        '''
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                dir("cd-repo/${DEPLOY_PATH}") {
                    script {
                        sh '''
                        echo "Deploying to AKS..."
                        kubectl apply -f . -n ${NAMESPACE}
                        kubectl rollout status deployment/myapp -n ${NAMESPACE} || true
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Success"
        }
        failure {
            echo "Failure"
        }
    }
}
