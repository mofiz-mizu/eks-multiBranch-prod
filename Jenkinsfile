pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "haquemofiz/multibranch-flask-app"
        GIT_USER   = "mofiz-mizu"
        GIT_EMAIL  = "haquemizu@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when { branch 'main' }
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        // stage('Update K8s Manifest') {
        //     when { branch 'main' }
        //     steps {
        //         script {
        //             withCredentials([usernamePassword(
        //                 credentialsId: 'github-creds',
        //                 usernameVariable: 'GIT_USERNAME',
        //                 passwordVariable: 'GIT_TOKEN'
        //             )]) {
        //                 sh '''
        //                   echo "Current Directory: $(pwd)"
        //                   echo "Listing all files recursively:"
        //                   ls -R
        //                 '''                        
        //                 sh """
        //                 set -e
        //                 git config user.name "$GIT_USER"
        //                 git config user.email "$GIT_EMAIL"

        //                 git fetch origin
        //                 git checkout main
        //                 git reset --hard origin/main

        //                 sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yml

        //                 git add k8s/deployment.yml
        //                 git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
        //                 git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/mofiz-mizu/eks-multiBranch-prod.git main
        //                 """
        //             }
        //         }
        //     }
        // }
        stage('Update K8s Manifest') {
            when { branch 'main' }
            steps {
                script {
                    // Ensure source code is present in this specific stage's workspace
                    checkout scm 

                    withCredentials([usernamePassword(credentialsId: 'github-creds', 
                                    usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        sh '''
                            set -e
                            git config user.name "${GIT_USER}"
                            git config user.email "${GIT_EMAIL}"

                            git fetch origin
                            git checkout main
                            git reset --hard origin/main

                            # Verify file exists before running sed
                            if [ -f "k8s/deployment.yml" ]; then
                                sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
                            else
                                echo "ERROR: k8s/deployment.yml not found!"
                                ls -R
                                exit 1
                            fi

                            git add k8s/deployment.yml
                            git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                            
                            # Use single quotes for the sh block to keep GIT_TOKEN secure
                            git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com main
                        '''
                    }
                }
            }
        }

    }
}
