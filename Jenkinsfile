pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven'
    }

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE_BE = "your-dockerhub-username/test-be"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // CI: 모든 브랜치에서 실행되는 테스트
        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    echo '테스트 성공! 코드가 품질 기준을 통과했습니다.'
                }
                failure {
                    echo '테스트 실패! 코드를 수정해주세요.'
                }
            }
        }

        // CD: main/master 브랜치에서만 실행되는 배포 단계
        stage('Package') {
            when {
                branch 'master' // 또는 'main'
            }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build & Push Docker Image') {
            when {
                branch 'master' // 또는 'main'
            }
            steps {
                // Docker Hub 로그인
                sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'

                // Docker 이미지 빌드 및 푸시
                sh "docker build -t ${DOCKER_IMAGE_BE}:${VERSION} -t ${DOCKER_IMAGE_BE}:latest ."
                sh "docker push ${DOCKER_IMAGE_BE}:${VERSION}"
                sh "docker push ${DOCKER_IMAGE_BE}:latest"
            }
        }

        stage('Trigger Deployment') {
            when {
                branch 'master' // 또는 'main'
            }
            steps {
                // 배포 작업 트리거
                build job: 'TEST-Deploy', wait: false,
                parameters: [
                    string(name: 'BE_VERSION', value: "${VERSION}"),
                    string(name: 'COMPONENT', value: "backend")
                ]
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo '백엔드 파이프라인이 성공적으로 완료되었습니다!'
        }
        failure {
            echo '백엔드 파이프라인이 실패했습니다.'
        }
    }
}