pipeline {
    agent any

    stages {
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
        always {
            script {
                if (currentBuild.result == 'UNSTABLE') {
                    githubNotify credentialsId: 'github-token-priyanshu',
                                 context: 'ci/jenkins',
                                 status: 'FAILURE',
                                 description: 'Some tests failed'
                } else {
                    githubNotify credentialsId: 'github-token-priyanshu',
                                 context: 'ci/jenkins',
                                 status: 'SUCCESS',
                                 description: 'All tests passed'
                }
            }
        }
    }
}
