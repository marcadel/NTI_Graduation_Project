pipeline {
    agent any 

    environment {
        REPO_SERVER = "637423360254.dkr.ecr.us-east-1.amazonaws.com/graduation"
        REPO_NAME_BACKEND = "${REPO_SERVER}:backend"
        REPO_NAME_FRONTEND = "${REPO_SERVER}:frontend"
        IMAGE_VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage("build image and push to ECR") {
            steps {
                script {
                    echo "building and pushing docker images ..."
                    withCredentials([
                        usernamePassword(credentialsId: 'ecr-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')
                    ]){
                        sh "docker login -u ${USER} -p ${PASS} ${REPO_SERVER}"
                        sh "docker build app/back/. -t ${REPO_NAME_BACKEND}-${IMAGE_VERSION}"
                        sh "docker push ${REPO_NAME_BACKEND}-${IMAGE_VERSION}"
                        sh "docker build app/front/. -t ${REPO_NAME_FRONTEND}-${IMAGE_VERSION}"
                        sh "docker push ${REPO_NAME_FRONTEND}-${IMAGE_VERSION}"
                    }
                }
            }
        }

        stage("change image version") {
            steps {
                script {
                    echo "change image version .."
                    sh "sed -i \"s|image:.*|image: ${REPO_NAME_BACKEND}-${IMAGE_VERSION}|g\" k8s/back.yml"
                    sh "sed -i \"s|image:.*|image: ${REPO_NAME_FRONTEND}-${IMAGE_VERSION}|g\" k8s/front.yml"
                }
            }
        }

        stage('Deploy to eks cluster') {
            environment {
                AWS_ACCESS_KEY_ID = credentials("aws_access_key_id")
                AWS_SECRET_ACCESS_KEY = credentials("aws_secret_access_key")
            }
            steps {
                echo 'Deploying to eks cluster ... '
                withCredentials([file(credentialsId:'kube-config', variable:'KUBECONFIG')]){
                    script{
                        sh 'kubectl apply -f k8s'
                    }
                }
            }
        }
    }
}