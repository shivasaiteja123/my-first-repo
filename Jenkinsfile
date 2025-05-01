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

        stage('Fetch SonarQube Results') {
            steps {
                echo 'Fetching analysis results...'
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
                        def body = "The quality gate result is: ${currentBuild.result}. Please review the analysis report."

                        def sendEmailCommand = """
                            curl -X POST 'https://api.mailgun.net/v3/${MAILGUN_DOMAIN}/messages' \
                            --user 'api:${MAILGUN_API_KEY}' \
                            -F from='${SENDER_EMAIL}' \
                            -F to='${recipient}' \
                            -F subject='${subject}' \
                            -F text='${body}'
                        """
                        bat sendEmailCommand
                    }
                }
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
