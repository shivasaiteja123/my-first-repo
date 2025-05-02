pipeline {
    agent any

    environment {
        SONARQUBE_TOKEN = credentials('sqa_64083ad40e70ff94ceac7d010ed6db5699df43e7')
        SONARQUBE_PROJECT_KEY = 'code-review'
        MAILGUN_API_KEY = credentials('2ce6a04c021ad1d678ec39b531e1e929-3d4b3a2a-4eff30ae')
        MAILGUN_DOMAIN = 'sandbox0dc4f69b76bb445480873a1165edcb68.mailgun.org' // Replace with your Mailgun domain
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/shivasaiteja123/my-first-repo.git'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = "${env.SONARQUBE_TOKEN}"
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat "\"C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner.bat\" -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} -Dsonar.sources=. -Dsonar.token=${SONAR_TOKEN}"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Fetch SonarQube Results') {
            steps {
                echo 'Fetching analysis results...'
                // You could add logic here to fetch metrics if needed.
            }
        }

        stage('Email Notification') {
            steps {
                withCredentials([string(credentialsId: 'mailgun-api-key', variable: 'MAILGUN_API_KEY')]) {
                    script {
                        def subject = "SonarQube Quality Gate: PASSED"
                        def body = "The project *${SONARQUBE_PROJECT_KEY}* has passed the quality gate. Visit: http://localhost:9000/dashboard?id=${SONARQUBE_PROJECT_KEY}"
                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "https://api.mailgun.net/v3/${MAILGUN_DOMAIN}/messages",
                            authentication: 'mailgun-api-key',
                            contentType: 'APPLICATION_FORM',
                            requestBody: "from=Jenkins CI <jenkins@${MAILGUN_DOMAIN}>&to=youremail@example.com&subject=${subject}&text=${body}"
                        )
                        echo "Email sent: ${response.status}"
                    }
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: '**/.scannerwork/**', allowEmptyArchive: true
            }
        }

        stage('Cleanup SonarQube') {
            steps {
                echo "Cleanup steps can be defined here if needed (e.g., deleting project via API)."
            }
        }
    }
}
