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
        DOCKER_IMAGE = 'hanjunn/hanjun-site'
        VERSION = "${BUILD_NUMBER}" // Jenkins 빌드 번호를 버전으로 사용
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
                container('docker') {
                    dir('.') { // 빌드 컨텍스트를 최상위 디렉토리로 변경
                        script {
                            def imageTag = "${DOCKER_IMAGE}:${VERSION}"
                            docker.build(imageTag, "-f docker/Dockerfile .")
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    script {
                        def imageTag = "${DOCKER_IMAGE}:${VERSION}"
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                            docker.image(imageTag).push()
                        }
                    }
                }
            }
        }

        stage('Update nginx-deployment.yaml') {
            steps {
                script {
                    def newImage = "${DOCKER_IMAGE}:${VERSION}"
                    // nginx-deployment.yaml 파일에서 image 태그를 새로운 이미지 버전으로 교체
                    sh """
                    sed -i 's|image: hanjunn/hanjun-site:latest|image: ${newImage}|g' /manifests/deployments/nginx-deployment.yaml
                    """
                }
            }
        }

        stage('Commit and Push nginx-deployment.yaml') {
            steps {
                container('jnlp') {
                    script {
                        // 변경된 nginx-deployment.yaml 파일을 git에 커밋하고 푸시
                        sh """
                        git config --global user.email "qwedfr79@naver.com"
                        git config --global user.name "hanjunnn"
                        git add /manifests/deployments/nginx-deployment.yaml
                        git commit -m "Update nginx deployment image version to ${VERSION}"
                        git push origin main
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build, push, and deployment file update successful!'
        }
        failure {
            echo 'Build or push failed.'
        }
    }
}
