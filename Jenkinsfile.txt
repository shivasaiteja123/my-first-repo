pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/shivasaiteja123/my-first-repo'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('code review') {
                    bat 'sonar-scanner.bat'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Send Email') {
            steps {
                bat '"C:\\Users\\varap\\AppData\\Local\\Programs\\Python\\Python313\\python.exe" "C:\\Python script\\send_mail.py"'
            }
        }
    }
}
