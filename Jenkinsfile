pipeline {
    agent any
    environment {
        GITNAME = 'hansususu'
        AWS_REGION = 'ap-northeast-2'
        GITMAIL = 'hansubin0039@gmail.com' 
        GITWEBADD = 'https://github.com/hansususu/nginx-test.git'
        GITSSHADD = 'git@github.com:hansususu/deployment'
        ECR_REGISTRY = '730335507432.dkr.ecr.ap-northeast-2.amazonaws.com'
        ECR_REPOSITORY = 'nginx'
        GITCREDENTIAL = 'git_cre'
        IMAGE_TAG = "${env.BUILD_ID}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [],
                userRemoteConfigs: [[credentialsId: GITCREDENTIAL, url: GITWEBADD]]])
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}")
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${ECR_REGISTRY}", '') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
         stage('EKS manifest file update') {
            steps {
                git credentialsId: GITCREDENTIAL, url: GITSSHADD, branch: 'main'
                sh "git config --global user.email ${GITMAIL}"
                sh "git config --global user.name ${GITNAME}"
                sh "sed -i 's@${DOCKERHUB}:.*@${ECR_REGISTRY}:${currentBuild.number}@g' nginx.yml"

                sh "git add ."
                sh "git branch -M main"
                sh "git commit -m 'fixed tag ${currentBuild.number}'"
                sh "git remote remove origin"
                sh "git remote add origin ${GITSSHADD}"
                sh "git push origin main"
            }
            post {
                failure {
                    sh "echo failed"
                }
                success {
                    sh "echo success"
                }
            }
        }
    }
}
