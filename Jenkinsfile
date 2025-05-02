pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'code-review'
        SENDER_EMAIL = 'saiteja.y@coresonant.com'
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
                script {
                    try {
                        timeout(time: 5, unit: 'MINUTES') {
                            def qualityGate = waitForQualityGate(abortPipeline: false)
                            if (qualityGate.status != 'OK') {
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    } catch (Exception e) {
                        echo "Quality Gate timed out or failed: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Fetch SonarQube Report') {
            steps {
                script {
                    echo "Fetching SonarQube analysis report..."
                    def sonarReportUrl = "http://localhost:9000/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}"
                    def sonarReportResponse = bat(script: "curl -u ${SONAR_AUTH_TOKEN}: ${sonarReportUrl}", returnStdout: true).trim()
                    writeFile(file: 'sonarqube_report.json', text: sonarReportResponse)
                    echo "SonarQube report fetched successfully."
                }
            }
        }

        stage('Email Notification') {
            steps {
                withCredentials([
                    string(credentialsId: 'MailgunAPI', variable: 'MAILGUN_API_KEY'),
                    string(credentialsId: 'Mailgundomain', variable: 'MAILGUN_DOMAIN')
                ]) {
                    script {
                        def recipient = 'yerramchattyshivasaiteja2003@gmail.com'
                        def subject = "SonarQube Analysis: Build ${currentBuild.result}"
                        def body = "The quality gate result is: ${currentBuild.result}. Please find the analysis report attached."

                        echo "Sending email to ${recipient} via domain ${MAILGUN_DOMAIN}"

                        def sendEmailCommand = """
                            curl -X POST "https://api.mailgun.net/v3/${MAILGUN_DOMAIN}/messages" ^
                            --user "api:${MAILGUN_API_KEY}" ^
                            -F from="${SENDER_EMAIL}" ^
                            -F to="${recipient}" ^
                            -F subject="${subject}" ^
                            -F text="${body}" ^
                            -F attachment=@sonarqube_report.json
                        """
                        bat sendEmailCommand
                    }
                }
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'sonarqube_report.json', fingerprint: true
                echo 'Report archived successfully.'
            }
        }

        stage('Cleanup') {
            steps {
                echo 'You can implement project cleanup logic here using SonarQube API if needed.'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
