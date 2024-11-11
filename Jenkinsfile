pipeline {
    agent any
    
    environment {
        // GitHub 인증 정보 (Jenkins에 저장된 자격 증명 ID 사용)
        GITHUB_CREDENTIALS = credentials('github-token')
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'hanjunn/hanjun-site:latest'
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    // GitHub에서 소스 코드 체크아웃
                    git credentialsId: 'github-token', url: 'https://github.com/hanjunnn/k8s-ci-cd.git'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Docker 빌드: docker 폴더 내의 Dockerfile 사용
                    // 'docker' 폴더로 이동하여 빌드
                    dir('docker') {
                        docker.build("${DOCKER_IMAGE}")
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Docker Hub에 로그인
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        // Docker 이미지를 Docker Hub에 푸시
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build and push to Docker Hub successful!'
        }
        
        failure {
            echo 'Build or push failed.'
        }
    }
}
