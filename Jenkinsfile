pipeline {
    agent {
        label 'slave-1'
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
                    credentialsId: 'test',
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
}
