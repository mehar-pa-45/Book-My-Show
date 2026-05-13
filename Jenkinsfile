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

        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app

                if [ -f package.json ]; then
                    echo "Installing dependencies..."
                    rm -rf node_modules package-lock.json
                    npm install
                else
                    echo "package.json not found!"
                    exit 1
                fi
                '''
            }
        }

        /* ================= SONARQUBE ================= */

        stage('SonarQube Analysis') {
            steps {
                script {
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
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        /* ================= OWASP ================= */

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    odcInstallation: 'DP-Check',
                    additionalArguments: '''
                    --scan bookmyshow-app
                    --exclude **/node_modules/**
                    '''
                )

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml'
                )
            }
        }

        /* ================= TRIVY ================= */

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        /* ================= DOCKER ================= */

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

        /* ================= DEPLOY ================= */

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
                docker ps -a
                docker logs bms
                '''
            }
        }
    }

    /* ================= EMAIL ================= */

    post {
        always {
            emailext(
                attachLog: true,
                subject: "${currentBuild.result}: ${env.JOB_NAME}",
                body: """
                Project: ${env.JOB_NAME}<br/>
                Build Number: ${env.BUILD_NUMBER}<br/>
                Build URL: ${env.BUILD_URL}<br/>
                """,
                to: 'prsam.789@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
