pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIAL_ID = 'mlops-jenkins-dockerhub-token'
        DOCKERHUB_REGISTRY = 'https://registry.hub.docker.com'
        DOCKERHUB_REPOSITORY = 'mdabusiyam/mlops-iris-app'
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo 'Cloning GitHub Repo...'
                    checkout scmGit(
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[
                            credentialsId: 'mlops-aws-ecs-cicd',
                            url: 'https://github.com/mdabusiyam/AWS-ECS-Jenkins-ML-CICD.git'
                        ]]
                    )
                }
            }
        }
        stage('Lint Code') {
            steps {
                script {
                    echo 'Linting Python Code...'
                    sh "python -m pip install --break-system-packages -r requirements.txt"
                    sh "pylint app.py train.py --output=pylint-report.txt --exit-zero"
                    sh "flake8 app.py train.py --ignore=E501,E302 --output-file=flake8-report.txt"
                    sh "black app.py train.py"
                }
            }
        }
        stage('Test Code') {
            steps {
                script {
                    echo 'Testing Python Code...'
                    sh "pytest tests/"
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                script {
                    echo 'Scanning Filesystem with Trivy...'
                    sh "trivy fs --format table -o trivy-fs-report.html ."
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker Image...'
                    dockerImage = docker.build("${DOCKERHUB_REPOSITORY}:latest") 
                }
            }
        }
        stage('Trivy Docker Image Scan') {
            steps {
                script {
                    echo 'Scanning Docker Image with Trivy...'
                    sh "trivy image --format table -o trivy-image-report.html ${DOCKERHUB_REPOSITORY}:latest"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Pushing Docker Image to DockerHub...'
                    docker.withRegistry("${DOCKERHUB_REGISTRY}", "${DOCKERHUB_CREDENTIAL_ID}") {
                        dockerImage.push('latest')
                    }
                }
            }
        }
stage('Deploy') {
            steps {
                script {
                    echo 'Deploying to production...'
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'mlops-awscredential'
                    ]]) {
                        sh "aws ecs update-service --cluster mlops-cluster --service mlops-service --force-new-deployment --region ap-southeast-2"
                    }
                }
            }
        }
    }
}
