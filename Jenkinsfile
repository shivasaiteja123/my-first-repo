pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'code-review'
SENDER_EMAIL = 'saiteja.y@coresonant.com' // Replace with your verified Mailgun sender email

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
                        bat "\"C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner.bat\" -Dsonar.projectKey=%SONAR_PROJECT_KEY% -Dsonar.sources=. -Dsonar.token=%SONAR_AUTH_TOKEN%"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
SENDER_EMAIL = 'saiteja.y@coresonant.com' // Replace with your verified Mailgun sender email

                }
            }
        }

        stage('Fetch SonarQube Results') {
            steps {
                withCredentials([string(credentialsId: 'code review', variable: 'SONAR_AUTH_TOKEN')]) {
                    script {
                        def apiUrl = "http://localhost:9000/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}"
                        def curlCommand = "curl -u ${SONAR_AUTH_TOKEN}: ${apiUrl} -o quality-gate.json"
                        bat curlCommand
                        archiveArtifacts artifacts: 'quality-gate.json', allowEmptyArchive: true
                    }
                }
            }
        }

        stage('Email Notification') {
            steps {
SENDER_EMAIL = 'saiteja.y@coresonant.com' // Replace with your verified Mailgun sender email

            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: '**/.scannerwork/report-task.txt', allowEmptyArchive: true
                echo 'Archiving results...'
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up SonarQube project if needed...'
                // Add deletion call to SonarQube API here if desired
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
