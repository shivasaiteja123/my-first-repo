pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'code-review'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/shivasaiteja123/my-first-repo.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'code review', variable: 'SONAR_AUTH_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        bat '"C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner.bat" -Dsonar.projectKey=%SONAR_PROJECT_KEY% -Dsonar.sources=. -Dsonar.token=%SONAR_AUTH_TOKEN%'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Fetch SonarQube Results') {
            steps {
                echo 'Fetching analysis results...'
            }
        }

        stage('Email Notification') {
            steps {
                echo 'Sending email report...'
            }
        }

        stage('Archive Reports') {
            steps {
                echo 'Archiving results...'
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up SonarQube project if needed...'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
