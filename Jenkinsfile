pipeline {

    agent {
        label 'slave-1'
    }

    environment {

        IMAGE_NAME = 'kalpuaggressive/hostelmanagement'

        IMAGE_TAG = 'v1'
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

        stage('Deploy WAR to Nexus') {

            steps {

                sh 'mvn deploy'

            }

        }

        stage('Archive Artifact') {

            steps {

                archiveArtifacts artifacts: 'target/*.war', fingerprint: true

            }

        }

        stage('Check Agent') {

            steps {

                sh '''

                whoami

                docker --version

                docker ps

                '''

            }

        }

        stage('Build Docker Image') {

            steps {

                sh '''

                docker build -t $IMAGE_NAME:$IMAGE_TAG .

                '''

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

                    docker push $IMAGE_NAME:$IMAGE_TAG

                    docker logout

                    '''

                }

            }

        }

        stage('Deploy to Tomcat') {

            steps {

                sh '''

                cp target/*.war /opt/tomcat/webapps/

                '''

            }

        }

    }

    post {

        success {

            echo 'Pipeline Completed Successfully'

        }

        failure {

            echo 'Pipeline Failed'

        }

        always {

            cleanWs()

        }

    }

}
