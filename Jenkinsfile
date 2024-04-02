pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('ishwari-aws-key')
        DOCKER_HUB_KEY = credentials('ishwari-docker-hub')
        DOCKER_IMAGE_RESUME_BUILDER_FRONTEND = 'misscoder21/resumebuilder-fe'
        DOCKER_IMAGE_RESUME_BUILDER_BACKEND = 'misscoder21/resumebuilder-be'
        GITHUB_URL = 'https://github.com/MissIshwari/ResumeAI'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('checkout') {
            steps {
                git branch: env.GIT_BRANCH, url: env.GITHUB_URL
            }
        }
        stage('create .env'){
            steps{
                script {
                    def envContent = """
                        MONGO_URL='mongodb+srv://group4:group4@cluster0.wbha2pq.mongodb.net/resume-builder'
                        JWT_SECRET_KEY="MYREALLYSECRETKEY"
                        OPENAI_KEY="OPENAI_API_KEY"
                        GMAIL_USER="THIS EMAIL IS USED TO SEND RESUMES"
                        GMAIL_PASS="PASSWORD USED BY NODEMAILER"
                        FRONT_END="http://localhost:80"
                    """
                    writeFile(file: './ResumeBuilderBackend/.env', text: envContent.trim())
                }
            }
        }
        stage('build images') {
            parallel {
                stage('build backend') {
                    steps {
                        script {
                            docker.build("${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID}", './ResumeBuilderBackend/')
                        }
                    }
                }
                stage('build frontend') {
                    steps {
                        script {
                            docker.build("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}", './ResumeBuilderAngular/')
                        }
                    }
                }
            }
        }
        stage('push images to docker') {
            steps {
                script {
                    sh 'docker logout'
                    withCredentials([usernamePassword(credentialsId: 'ishwari-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo -n ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    }
                    withCredentials([usernamePassword(credentialsId: 'ishwari-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        docker.withRegistry('https://index.docker.io/v1/', 'ishwari-docker-hub') {
                            docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID}").push()
                            docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}").push()
                        }
                    }
                }
            }
        }
        stage('connect to eks'){
            steps{
                script{
                    withCredentials([aws(credentialsId: 'ishwari-aws-key', region:'eu-west-2')]) {
                        sh 'echo hi'
                        def eksClusterExists = sh(script: 'aws eks describe-cluster --name ishwari-eks-cluster-1 --region eu-west-2', 
                        returnStatus: true) == 0
                        if(!eksClusterExists){
                            sh '''
                            eksctl create cluster --name ishwari-eks-cluster-1 --region eu-west-2 --nodegroup-name standard-workers --node-type t3.micro --nodes 2 --nodes-min 1 --nodes-max 3
                            '''
                        }
                    }
                }
            }
        }
        stage('deploy to eks'){
            steps{
                script{
                    withCredentials([aws(credentialsId: 'ishwari-aws-key', region:'eu-west-2')]) {
                        sh '''
                        aws eks --region eu-west-2 update-kubeconfig --name ishwari-eks-cluster-1
                        kubectl version
                        cd ResumeBuilderBackend/
                        kubectl apply -f backend-deployment.yaml
                        kubectl apply -f backend-service.yaml
                        '''
                    }
                }
            }
        }
    }
}
