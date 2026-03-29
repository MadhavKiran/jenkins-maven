pipeline {
    agent { label 'maven-agent' }

    environment {
        APP_SERVER_IP   = '172.31.76.1'
        APP_SERVER_USER = 'ubuntu'
        DEPLOY_DIR      = '/opt/application'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Security Scan - Trivy') {
            steps {
                sh '''
		   trivy fs --exit-code 1 --severity CRITICAL \
                     --format table .
                       '''
                
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar',
                    fingerprint: true
            }
        }

        stage('Deploy to App Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(credentials: ['app-server-ssh']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no \
                            target/*.jar \
                            ubuntu@''' + env.APP_SERVER_IP + ''':/opt/application/app.jar
                    '''
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            ubuntu@''' + env.APP_SERVER_IP + ''' \
                            "cd /opt/application && nohup java -jar app.jar > app.log 2>&1 &"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
