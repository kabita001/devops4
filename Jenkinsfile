pipeline {
    agent any

    tools {
        nodejs 'NodeJS 20.18.0'
    }

    environment {
        SCANNER_HOME = tool 'SonarQube Scanner 6.2.1.4610'

        NODEJS_HOME = tool name: 'NodeJS 20.18.0'
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"

        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_CREDENTIALS_ID = 'NexusCreds'

        SONAR_HOST_URL = "http://172.23.224.1:9000"
    }


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Node Installation') {
            steps {
                sh 'node --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                withCredentials([file(credentialsId: 'NexusCreds', variable: 'kabitanpmrc')]) {
                    echo 'Building...'
                    sh "npm install --userconfig $kabitanpmrc --registry http://172.23.224.1:8081/repository/devops4-proxy/ --loglevel verbose"
                }
            }
        }

        stage('Build') {
            steps {
                withCredentials([file(credentialsId: 'NexusCreds', variable: 'kabitanpmrc')]) {
                    echo 'Building...'
                    sh "npm run build --userconfig $kabitanpmrc --registry http://172.23.224.1:8081/repository/devops4-proxy/ --loglevel verbose"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv() {
                        sh """${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://172.23.224.1:9000 \
                        -Dsonar.token=sqp_2f96e2e144a9de8c9bdb3c53da4de5d9e29afe61 \
                        -Dsonar.exclusions=**/node_modules/** \
                        -Dsonar.projectKey=devops4"""
                    }
                }
            }
        }

        stage('Verifying Quality Gate') {
            steps {
                script {
                    sleep(time: 30, unit: 'SECONDS')
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status == 'IN_PROGRESS') {
                        sleep(time: 30, unit: 'SECONDS')
                        error "In progress. Trying again..."
                    }

                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                    else {
                        echo "Quality Gate passed: ${qualityGate.status}"
                    }
                }
            }
        }


        stage('Publishing to Nexus') {
            steps {
                withCredentials([file(credentialsId: 'NexusCreds', variable: 'kabitanpmrc')]) {
                    echo 'Publishing to Nexus...'
                    sh "npm publish --userconfig ${kabitanpmrc} --loglevel verbose"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}