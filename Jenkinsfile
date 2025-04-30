pipeline {
    agent any
    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = 'sqa_64083ad40e70ff94ceac7d010ed6db5699df43e7'
        REPO_URL = 'https://github.com/shivasaiteja123/my-first-repo.git'
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Manually checkout the repository
                    git url: "${REPO_URL}"
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        bat '''
                        C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner.bat -Dsonar.projectKey=code-review -Dsonar.sources=. -Dsonar.token=${SONAR_AUTH_TOKEN}
                        '''
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    waitForQualityGate(abortPipeline: true)
                }
            }
        }
        stage('Fetch SonarQube Results') {
            steps {
                echo 'Fetching SonarQube results'
                // You could add logic to fetch detailed results from the SonarQube API here.
            }
        }
        stage('Email Notification') {
            steps {
                echo 'Sending email notification'
                // Trigger the Python email script here, for example:
                bat '"C:\\Users\\varap\\AppData\\Local\\Programs\\Python\\Python313\\python.exe" "C:\\Python script\\send_mail.py"'
            }
        }
        stage('Archive Reports') {
            steps {
                echo 'Archiving reports'
                // Archive SonarQube analysis or build logs here.
            }
        }
        stage('Cleanup') {
            steps {
                echo 'Cleanup'
                // Clean up workspace or logs if needed
            }
        }
        stage('Post Actions') {
            steps {
                echo 'Pipeline completed.'
            }
        }
    }
    post {
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
