pipeline {
    agent any

    environment {
        APP_NAME = "devsecops-demo"
        DOCKERHUB_USERNAME = "akshaykothe"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        SCANNER_HOME = tool 'SonarQube-Scanner'
        MANIFEST_REPO = "https://github.com/Akshaykothe/devsecops-manifests.git"
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
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        bat "%SCANNER_HOME%\\bin\\sonar-scanner -Dsonar.projectKey=devsecops-demo -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=%SONAR_TOKEN%"
                    }
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

        stage('Update Manifest Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    bat """
                        git clone https://%GIT_USER%:%GIT_PASS%@github.com/Akshaykothe/devsecops-manifests.git
                        cd devsecops-manifests
                        powershell -Command "(Get-Content deployment.yaml) -replace 'akshaykothe/devsecops-demo:.*', 'akshaykothe/devsecops-demo:%BUILD_NUMBER%' | Set-Content deployment.yaml"
                        git config user.email "jenkins@devsecops.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image to %IMAGE_NAME%:%BUILD_NUMBER%"
                        git push https://%GIT_USER%:%GIT_PASS%@github.com/Akshaykothe/devsecops-manifests.git main
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS - Image pushed and manifest updated!"
        }
        failure {
            echo "Pipeline FAILED"
        }
    }
}