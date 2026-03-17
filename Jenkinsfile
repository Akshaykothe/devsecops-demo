pipeline {
    agent any

    environment {
        APP_NAME = "devsecops-demo"
        DOCKERHUB_USERNAME = "akshaykothe"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        SCANNER_HOME = tool 'SonarQube-Scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'pytest test_app.py -v'
            }
        }

        stage('SonarQube SAST') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat "%SCANNER_HOME%\\bin\\sonar-scanner -Dsonar.projectKey=devsecops-demo -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqa_eb9a6f217cf218bce9c842bcce808030015be49b"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%BUILD_NUMBER% ."
            }
        }

        stage('Trivy Security Scan') {
            steps {
                bat "C:\\Users\\Akshay-ragina-kothe\\AppData\\Local\\Microsoft\\WinGet\\Links\\trivy.exe image --exit-code 0 --severity HIGH,CRITICAL %IMAGE_NAME%:%BUILD_NUMBER%"
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push %IMAGE_NAME%:%BUILD_NUMBER%"
                    bat "docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest"
                    bat "docker push %IMAGE_NAME%:latest"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS - Image pushed to DockerHub!"
        }
        failure {
            echo "Pipeline FAILED"
        }
    }
}