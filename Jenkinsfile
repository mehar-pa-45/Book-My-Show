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
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mehar-pa-45/Book-My-Show.git'
                sh 'ls -la'
            }
        }

        /* ---------------- INSTALL DEPENDENCIES ---------------- */

        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app

                if [ -f package.json ]; then
                    rm -rf node_modules package-lock.json
                    npm install
                else
                    echo "package.json not found!"
                    exit 1
                fi
                '''
            }
        }

        /* ---------------- SONARQUBE ANALYSIS ---------------- */

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube-Scanner') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectName=BMS \
                        -Dsonar.projectKey=BMS \
                        -Dsonar.sources=bookmyshow-app
                        """
                    }
                }
            }
        }

        /* ---------------- QUALITY GATE ---------------- */

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }

        /* ---------------- OWASP SCAN ---------------- */

        stage('OWASP Dependency Check') {
    steps {
        dependencyCheck(
            odcInstallation: 'DP-Check',
            additionalArguments: '''
            --scan bookmyshow-app
            --exclude **/node_modules/**
            --format XML
            '''
        )

        dependencyCheckPublisher(
            pattern: '**/dependency-check-report.xml'
        )
    }
}

        /* ---------------- TRIVY SCAN ---------------- */

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        /* ---------------- DOCKER BUILD ---------------- */

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                        echo "Building Docker image..."
                        docker build --no-cache \
                        -t $DOCKER_IMAGE \
                        -f bookmyshow-app/Dockerfile bookmyshow-app

                        echo "Pushing Docker image..."
                        docker push $DOCKER_IMAGE
                        '''
                    }
                }
            }
        }

        /* ---------------- DEPLOY CONTAINER ---------------- */

        stage('Deploy Container') {
            steps {
                sh '''
                echo "Stopping old container..."
                docker stop bms || true
                docker rm bms || true

                echo "Running new container..."
                docker run -d \
                --name bms \
                --restart=always \
                -p 3000:3000 \
                $DOCKER_IMAGE

                sleep 10
                docker logs bms
                '''
            }
        }
    }

    /* ---------------- POST ACTION ---------------- */

    post {
        always {
            emailext attachLog: true,
                subject: "${currentBuild.result}",
                body: """
                Project: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """,
                to: 'msan8795@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
        }
    }
}
