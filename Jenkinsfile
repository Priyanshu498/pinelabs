pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
        always {
            script {
                def status = (currentBuild.result == 'UNSTABLE') ? "FAILURE" : "SUCCESS"

                githubNotify credentialsId: 'priyanshu',
                             repo: 'pinelabs',
                             account: 'Priyanshu498',
                             context: 'ci/jenkins',
                             status: status,
                             description: 'Jenkins CI result',
                             sha: env.GIT_COMMIT
            }
        }
    }
}
