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
            environment {
                SONARQUBE = credentials('code review')
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat "sonar-scanner.bat -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.sources=. -Dsonar.token=%SONAR_AUTH_TOKEN%"
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
                echo 'Fetching SonarQube analysis results...'
            }
        }

        stage('Email Notification') {
            steps {
                echo 'Sending email notification...'
            }
        }

        stage('Archive Reports') {
            steps {
                echo 'Archiving reports...'
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up...'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
