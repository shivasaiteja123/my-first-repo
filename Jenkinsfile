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
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate(abortPipeline: false)
                        def gateStatus = (qg.status == 'OK') ? 'Passed' : 'Failed'
                        currentBuild.result = (qg.status == 'OK') ? 'SUCCESS' : 'UNSTABLE'

                        withCredentials([
                            string(credentialsId: 'MailgunAPI', variable: 'MG_API'),
                            string(credentialsId: 'Mailgundomain', variable: 'MG_DOMAIN')
                        ]) {
                            def recipient = 'yerramchattyshivasaiteja2003@gmail.com'
                            def mailSubject = "SonarQube Analysis: Build ${currentBuild.result}"

                            // Update these two URLs with your current Ngrok links
                            def sonarQubeUrl = "https://your-ngrok-sonarqube-url.ngrok-free.app/dashboard?id=${SONAR_PROJECT_KEY}"
                            def jenkinsUrl = "https://2fe7-183-82-120-202.ngrok-free.app/job/Jenkinsfile/"

                            def mailBody = """\
Quality Gate Result: ${gateStatus}.
SonarQube Report: ${sonarQubeUrl}
Jenkins Build: ${jenkinsUrl}
"""

                            writeFile file: 'sendMail.bat', text: """
curl -s --user "api:${MG_API}" https://api.mailgun.net/v3/${MG_DOMAIN}/messages ^
  -F from="${SENDER_EMAIL}" ^ 
  -F to="${recipient}" ^ 
  -F subject="${mailSubject}" ^ 
  -F text="${mailBody}"
"""

                            def status = bat(script: 'call sendMail.bat', returnStatus: true)
                            if (status != 0) {
                                echo "Warning: sendMail.bat failed with exit code ${status}, but continuing the pipeline."
                            }
                        }
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
                echo 'Cleaning up...'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}
