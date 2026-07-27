pipeline {
    agent {
        label 'slave-1'
    }

    tools {
        maven 'Maven'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Kalpeshpatilblaze/HostelManagment1.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=HostelManagement \
                    -Dsonar.projectName=HostelManagement
                    '''
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

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Check Agent User') {
            steps {
                sh '''
                echo "Current User:"
                whoami

                echo "User Details:"
                id

                echo "Docker Socket:"
                ls -l /var/run/docker.sock

                echo "Docker Version:"
                docker --version

                echo "Docker Test:"
                docker ps
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t kalpuaggressive/hostelmanagement:v1 .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerub-id',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push kalpuaggressive/hostelmanagement:v1
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh 'cp target/*.war /opt/tomcat/webapps/'
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
