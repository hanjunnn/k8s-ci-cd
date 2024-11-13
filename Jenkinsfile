pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yamlFile 'agentpod.yaml'
        }
    }

    environment {
        GITHUB_CREDENTIALS = credentials('github-token')
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE = 'hanjunn/hanjun-site:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                container('jnlp') {
                    script {
                        git credentialsId: 'github-token', url: 'https://github.com/hanjunnn/k8s-ci-cd.git', branch: 'main'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') { // docker 컨테이너를 사용
                    script {
                        // 상위 디렉토리에서 빌드 컨텍스트로 설정하고 docker 폴더 안의 Dockerfile을 사용
                        dir('docker') {
                            script {
                                docker.build("${DOCKER_IMAGE}", "..") // 상위 디렉터리를 빌드 컨텍스트로 지정
                            }
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                container('docker') { // docker 컨테이너를 사용
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                            docker.image("${DOCKER_IMAGE}").push()
                        }
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
