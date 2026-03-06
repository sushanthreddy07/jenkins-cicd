pipeline {
    agent { label 'slave' }   // change to your actual agent label if different

    environment {
        APP_SERVER = '172.31.69.45'   // your app server private IP
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${env.BRANCH_NAME}",
                    credentialsId: 'github-creds',
                    url: 'https://github.com/sushanthreddy07/jenkins-cicd.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    echo "Running security scan..."
                    mkdir -p scan-report
                    mvn dependency:tree > scan-report/dependency-report.txt
                '''
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar, target/*.war', fingerprint: true
            }
        }

        stage('Deploy to Application Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(credentials: ['appserver-ssh']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no target/*.jar ubuntu@${APP_SERVER}:/home/ubuntu/
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} '
                            pkill -f "java -jar" || true
                            nohup java -jar /home/ubuntu/*.jar > /home/ubuntu/app.log 2>&1 &
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
