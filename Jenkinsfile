pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Priyanshu498/pinelabs.git'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running dummy tests..."
            }
        }

        stage('Publish Test Report') {
            steps {
                junit 'reports/junit.xml'
            }
        }
    }

    post {
        success {
            githubNotify credentialsId: 'github-token-priyanshu',
                         context: 'ci/jenkins',
                         status: 'SUCCESS',
                         description: 'All tests passed'
        }

        failure {
            githubNotify credentialsId: 'github-token-priyanshu',
                         context: 'ci/jenkins',
                         status: 'FAILURE',
                         description: 'Some tests failed'
        }
    }
}
