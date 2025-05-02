pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'code-review'
        SENDER_EMAIL = 'saiteja.y@coresonant.com' // 🔁 Replace with your verified Mailgun sender email
        SONAR_HOST_URL = 'http://localhost:9000' // Update with your SonarQube instance URL
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
                script {
                    echo 'Fetching SonarQube analysis results...'
                    // Make a REST API call to SonarQube to get the analysis details
                    def sonarApiUrl = "${SONAR_HOST_URL}/api/issues/search?componentKeys=${SONAR_PROJECT_KEY}&resolved=false"
                    def jsonResponse = sh(script: """
                        curl -s -u ${SONAR_AUTH_TOKEN}: ${sonarApiUrl}
                    """, returnStdout: true).trim()
                    
                    def json = readJSON text: jsonResponse
                    def bugs = json.issues.count { it.type == 'BUG' }
                    def vulnerabilities = json.issues.count { it.type == 'VULNERABILITY' }
                    def codeSmells = json.issues.count { it.type == 'CODE_SMELL' }

                    // Fetch coverage data
                    def coverageData = sh(script: """
                        curl -s -u ${SONAR_AUTH_TOKEN}: ${SONAR_HOST_URL}/api/measures/component?component=${SONAR_PROJECT_KEY}&metricKeys=coverage
                    """, returnStdout: true).trim()
                    def coverageJson = readJSON text: coverageData
                    def coverage = coverageJson.component.measures[0].value

                    // Fetch duplicated lines
                    def duplicationData = sh(script: """
                        curl -s -u ${SONAR_AUTH_TOKEN}: ${SONAR_HOST_URL}/api/measures/component?component=${SONAR_PROJECT_KEY}&metricKeys=duplicated_lines_density
                    """, returnStdout: true).trim()
                    def duplicationJson = readJSON text: duplicationData
                    def duplicatedCode = duplicationJson.component.measures[0].value

                    // Set environment variables for email
                    env.BUGS = bugs
                    env.VULNERABILITIES = vulnerabilities
                    env.CODE_SMELLS = codeSmells
                    env.COVERAGE = coverage
                    env.DUPLICATED_CODE = duplicatedCode
                }
            }
        }

        stage('Email Notification') {
            steps {
                withCredentials([string(credentialsId: 'MailgunAPI', variable: 'MAILGUN_API_KEY'),
                                 string(credentialsId: 'Mailgundomain', variable: 'MAILGUN_DOMAIN')]) {
                    script {
                        def recipient = 'yerramchattyshivasaiteja2003@gmail.com'
                        def subject = "SonarQube Analysis: Build ${currentBuild.result}"

                        // Create the email body with detailed analysis information
                        def body = """
                            The quality gate result is: ${currentBuild.result}.
                            Bugs: ${env.BUGS}
                            Vulnerabilities: ${env.VULNERABILITIES}
                            Code Smells: ${env.CODE_SMELLS}
                            Coverage: ${env.COVERAGE}%
                            Duplicated Code: ${env.DUPLICATED_CODE}%
                            
                            Please review the analysis report in SonarQube for further details.
                        """

                        echo "Sending email to ${recipient} via domain ${MAILGUN_DOMAIN}"

                        def sendEmailCommand = """
                            curl -X POST "https://api.mailgun.net/v3/${MAILGUN_DOMAIN}/messages" ^
                            --user "api:${MAILGUN_API_KEY}" ^
                            -F from="${SENDER_EMAIL}" ^
                            -F to="${recipient}" ^
                            -F subject="${subject}" ^
                            -F text="${body}"
                        """
                        bat sendEmailCommand
                    }
                }
            }
        }

        stage('Archive Reports') {
            steps {
                // Archiving any reports that you want to keep for later review (e.g., SonarQube report)
                archiveArtifacts '**/sonar-report.html'
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up SonarQube project if needed...'
                // Add any cleanup actions required for your SonarQube project or workspace
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
