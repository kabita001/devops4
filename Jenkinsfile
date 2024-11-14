pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS' // Assumes NodeJS is configured in Jenkins tools
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
        SONARQUBE_SERVER = 'SonarQube'  // Name configured for SonarQube in Jenkins
        NEXUS_URL = 'http://nexus:8081' // Nexus URL
        NEXUS_REPO = 'devops4'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials-id' // Jenkins credentials ID for Nexus
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'  // Assumes build command is defined in package.json
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner -D "sonar.projectKey=devops4" -D"sonar.sources=." -D"sonar.host.url=http://localhost:9000" -D"sonar.token=sqp_ed08b1684b41bf74704052d4fa78f39040e4f788"'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            }
        }

        // stage('Deploy to Nexus') {
        //     steps {
        //         nexusArtifactUploader(
        //             nexusVersion: 'nexus3',             
        //             protocol: 'http',                   
        //             nexusUrl: "${NEXUS_URL}",           
        //             repository: "${NEXUS_REPO}",        
        //             credentialsId: "${NEXUS_CREDENTIALS_ID}", 
        //             groupId: 'com.example',             
        //             version: '1.0.0',                   
        //             artifacts: [
        //                 [
        //                     artifactId: 'my-app',
        //                     classifier: '',
        //                     file: 'dist/my-app.zip',    
        //                     type: 'zip'
        //                 ]
        //             ]
        //         )
        //     }
        // }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

