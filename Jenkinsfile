pipeline {
    agent any

    tools {
        jdk 'jdk'
        nodejs 'nodejs'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'mehardocker45/bms:latest'
    }

    stages {

        stage('Clean Workspace') {
            steps { cleanWs() }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/mehar-pa-45/Book-My-Show.git'
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
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=BMS \
                    -Dsonar.projectKey=BMS \
                    -Dsonar.sources=bookmyshow-app \
                    -Dsonar.exclusions=**/node_modules/**
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        echo "Quality Gate Status: ${qg.status}"
                    }
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    odcInstallation: 'DP-Check',
                    additionalArguments: '--scan bookmyshow-app'
                )

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml'
                )
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withDockerRegistry(credentialsId: 'docker') {
                    sh '''
                    docker build -t $DOCKER_IMAGE -f bookmyshow-app/Dockerfile bookmyshow-app
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop bms || true
                docker rm bms || true
                docker run -d -p 3000:3000 --name bms $DOCKER_IMAGE
                '''
            }
        }
    }

    post {
        always {
            emailext(
                subject: "${currentBuild.result}",
                body: "Build URL: ${env.BUILD_URL}",
                to: 'prsam.789@gmail.com'
            )
        }
    }
}
