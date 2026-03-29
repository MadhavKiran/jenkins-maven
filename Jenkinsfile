pipeline {
    agent { label 'slave' }

    environment {
        APP_SERVER_IP = '172.31.76.1'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: env.BRANCH_NAME,
                    credentialsId: 'github-cred',
                    url: 'https://github.com/MadhavKiran/jenkins-maven.git'
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
                sh 'trivy fs --exit-code 1 --severity CRITICAL --format table .'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to App Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['app-server-ssh']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no target/*.jar ubuntu@172.31.76.1:/opt/application/app.jar
                        ssh -o StrictHostKeyChecking=no ubuntu@172.31.76.1 "pkill -f app.jar || true && nohup java -jar /opt/application/app.jar > /opt/application/app.log 2>&1 &"
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
