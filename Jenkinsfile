pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker-credentials'
        SONAR_CREDENTIALS_ID = 'sonar-token'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/bgharsalli/wordsmith.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([string(credentialsId: SONAR_CREDENTIALS_ID, variable: 'SONAR_TOKEN')]) {
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectKey=Wordsmith \
                            -Dsonar.projectName=Wordsmith \
                            -Dsonar.sources=./api \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push Api') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker build -t wordsmith-api ./api/'
                        sh 'docker tag wordsmith-api bilel216/wordsmith-api:latest'
                        sh 'docker push bilel216/wordsmith-api:latest'
                    }
                }
            }
        }

        stage('Docker Build & Push Web') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker build -t wordsmith-web ./web/'
                        sh 'docker tag wordsmith-web bilel216/wordsmith-web:latest'
                        sh 'docker push bilel216/wordsmith-web:latest'
                    }
                }
            }
        }

        stage('Trivy Images Scan') {
            steps {
                // Api Image Scan
                sh 'trivy image bilel216/wordsmith-api:latest > trivy-api-image.txt'
                // Web Image Scan
                sh 'trivy image bilel216/wordsmith-web:latest > trivy-web-image.txt'
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'microk8s kubectl apply -k .'
                }
            }
        }
    }
}
