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
                    withDockerRegistry(credentialsId: '30a2e9b9-a57c-417a-8e14-dd7e74745fb9', toolName: 'docker') {
                        sh "docker build -t hotstar ."
                        sh "docker tag hotstar krishnadale/test:latest"
                        sh "docker push krishnadale/test:latest"
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    sh 'trivy image --severity HIGH,CRITICAL gashok13193/test:latest --format table --output trivy-image-report.txt'
                }
            }
        }

        stage('Deploy Docker') {
            steps {
                sh "docker run --rm -d --name hotstar -p ${PORT}:3000 gashok13193/test:latest"
            }
        }
    }
}
