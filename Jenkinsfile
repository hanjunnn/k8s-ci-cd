pipeline {
    agent any
    
    environment {
        // GitHub 인증 정보 (Jenkins에 저장된 자격 증명 ID 사용)
        GITHUB_CREDENTIALS = credentials('github-token')
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE = 'hanjunn/hanjun-site:latest'
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    // GitHub에서 소스 코드 체크아웃
                    git credentialsId: 'github-token', url: 'https://github.com/hanjunnn/k8s-ci-cd.git', branch: 'main'
                }
            }
        }
        
        stage('Build Podman Image') {
            steps {
                script {
                    // Podman 빌드: docker 폴더 내의 Dockerfile 사용
                    // 'docker' 폴더로 이동하여 빌드
                    dir('docker') {
                        sh "podman build -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Docker Hub에 로그인
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Podman을 사용하여 Docker Hub에 로그인
                        sh "podman login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        
                        // Podman 이미지를 Docker Hub에 푸시
                        sh "podman push ${DOCKER_IMAGE}"
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
