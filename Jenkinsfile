pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME="clinkratcheat"
        APP_NAME="gitops-arco-app"
        IMAGE_TAG="${BUILD_NUMBER}"
        IMAGE_NAME="${DOCKERHUB_USERNAME}"+"/"+"${APP_NAME}"
        REGISTRY_CREDS='dockerhub'
    }
    stages { 
        stage('Cleanup workspace') {
            steps {
                script{
                    cleanWs()
                 }
            }
        }
        stage('Checkout SCM'){
            steps{
                script{
                    git creadentialsId : 'github',
                    url: 'https://github.com/jeet004a/EKS-CICD',
                    branch: 'master'
                }
            }
        }
        stage('Build Docker Image'){
            steps{
                script{
                    docker_image= docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps{
                script{
                    docker.withRegistry('',REGISTRY_CREDS){
                        docker_image.push("$BUILD_NUMBER")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage('Delete Docker images'){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Updating k8s deployment file'){
            steps{
                script{
                    sh """
                        cat deployment.yml
                        sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml
                        cat deployment.yml
                    """
                }
            }
        }
        stage('Push the changed deployment file to git'){
            steps{
                script{
                    sh """
                        git config --global user.name "jeet004a"
                        git config --global user.email "wastijeet110@gmail.com"
                        git add deployment.yml
                        git commit -m "Update deployment file"
                    """
                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        sh "git push https://github.com/jeet004a/EKS-CICD.git master" 
                    }
                }
            }
        }
    }
    
}

//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