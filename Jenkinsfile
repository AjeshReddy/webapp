pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="882956824445"
        AWS_DEFAULT_REGION="ap-northeast-1"
        IMAGE_REPO_NAME="arya"
        DOCKER_HUB_ID="ajesh22"
        IMAGE_TAG="4.0"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    stages {
        stage('Logging into AWS ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
            }
        }
        stage('clone the repo from github') {
            steps {
                git 'https://github.com/AjeshReddy/webapp.git'
            }
        }
        stage('maven build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('archive the artifacts:war') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
                sh "mv target/*war ."
            }
        }
        // install docker and docker pipeline plugins in jenkins
        stage('docker image build') {
            steps {
                sh "docker image build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
            }
        }
        //create a repo in ecr
        stage('Pushing to ECR') {
            steps {  
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
        }
        stage('Pushing to Docker-Hub') {
            steps {  
                withCredentials([string(credentialsId: 'password', variable: 'kubeadm')]) {
                sh "docker login -u ${DOCKER_HUB_ID} -p ${kubeadm}"
                }
                        
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${DOCKER_HUB_ID}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                sh "docker push ${DOCKER_HUB_ID}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
        }
        //install sshagent plugin and generate syntax in syntax generator: add user name and private ip of kubeadm master
        stage('k8s deploy') {
            steps {
                sshagent(['ajesh2217']) {
                                      
                    sh "scp -o StrictHostKeyChecking=no deployment-webapp.yml service-webapp-np.yml ubuntu@13.114.230.143:/home/ubuntu/"
                    script{
                        try{
                            sh "ssh ubuntu@13.114.230.143 sudo kubectl apply -f ."
                        }
                        catch(error){
                            sh "ssh ubuntu@13.114.230.143 sudo kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}
