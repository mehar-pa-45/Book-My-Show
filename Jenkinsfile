pipeline {
    agent any

    tools {
        nodejs 'nodejs'
        jdk 'jdk'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/mehar-pa-45/Book-My-Show.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app
                npm install
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Scanner') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=BMS \
                    -Dsonar.projectKey=BMS \
                    -Dsonar.sources=bookmyshow-app \
                    -Dsonar.exclusions=**/node_modules/**
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure"
                        }
                    }
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan bookmyshow-app',
                odcInstallation: 'dependency-check'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh '''
                trivy fs bookmyshow-app
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                        docker build -t mehardocker45/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app
                        docker push mehardocker45/bms:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop bms || true
                docker rm bms || true
                docker run -d -p 3000:3000 --name bms mehardocker45/bms:latest
                '''
            }
        }
    }

    post {
        always {
            emailext(
                to: 'prsam.789@gmail.com',
                subject: "Jenkins Pipeline Status",
                body: "Build Completed: ${currentBuild.currentResult}"
            )
        }
    }
}
