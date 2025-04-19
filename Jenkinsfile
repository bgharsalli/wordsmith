pipeline {
    agent any

    /*
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    */

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker'
        SONAR_CREDENTIALS_ID = 'Sonar-token'
    }
    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/mtbinds/wordsmith.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Wordsmith \
                        -Dsonar.projectKey=Wordsmith'''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: SONAR_CREDENTIALS_ID
                }
            }
        }

        /*
        stage('Install Dependencies') {
            steps {
                sh ''
                sh 'npm install'
            }
        }

        */

        //stage('OWASP FS SCAN') {
        //    steps {
        //        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //    }
        //}


        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push Api') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker build -t wordsmith-api .'
                        sh 'docker tag wordsmith-api madjidtaoualit/wordsmith-api:latest'
                        sh 'docker push madjidtaoualit/wordsmith-api:latest'
                    }
                }
            }
        }
        
        stage('Docker Build & Push Web') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker build -t wordsmith-web .'
                        sh 'docker tag wordsmith-web madjidtaoualit/wordsmith-web:latest'
                        sh 'docker push madjidtaoualit/wordsmith-web:latest'
                    }
                }
            }
        }

        stage('Trivy Images Scan') {
            steps {
                // Api Image Scan
                sh 'trivy image madjidtaoualit/wordsmith-api:latest > trivy-api-image.txt'
                // Web Image Scan
                sh 'trivy image madjidtaoualit/wordsmith-web:latest > trivy-web-image.txt'
            }
        }
        
        /*
        // Test Deploy with Docker
        stage('Deploy to Container') {
            steps {
                sh 'docker run -d --name wordsmith -p 8081:80 madjidtaoualit/wordsmith:latest'
            }
        }
        */

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'microk8s kubectl apply -k .'
                }
            }
        }
    }
}