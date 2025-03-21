pipeline {
    agent any
    tools {
        jdk 'jdk'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        PORT = '9000'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/krishnadale007/HotStar-clone.git'
            }
        }

        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Hotstar-clone \
                    -Dsonar.projectKey=Hotstar-clone \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP DEPENDENCY SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                script {
                    sh 'trivy fs --severity HIGH,CRITICAL ./ --format table --output trivy-fs-report.txt'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'a2eda937-40ba-4e5c-99db-5e70a09e16ac', toolName: 'docker') {
                        sh "docker build -t hotstar ."
                        sh "docker tag hotstar krishnadale/hotstar:latest"
                        sh "docker push krishnadale/hotstar:latest"
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    sh 'trivy image --severity HIGH,CRITICAL krishnadale/hotstar:latest --format table --output trivy-image-report.txt'
                }
            }
        }

        stage('Deploy Docker') {
            steps {
                sh "docker run --rm -d --name hotstar -p ${PORT}:3000 krishnadale/hotstar:latest"
            }
        }
    }
}
