pipeline {
    agent any

    tools {
        nodejs 'nodejs'
        jdk 'jdk'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        ODC_DATA = "/var/jenkins_home/odc-data"
    }

    stages {

        // ---------------- CLEAN ----------------
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        // ---------------- CHECKOUT ----------------
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/mehar-pa-45/Book-My-Show.git', branch: 'main'
            }
        }

        // ---------------- INSTALL ----------------
        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app
                npm install
                '''
            }
        }

        // ---------------- SONARQUBE ----------------
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

        // ---------------- QUALITY GATE ----------------
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

        // ---------------- OWASP (FAST VERSION) ----------------
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    odcInstallation: 'dependency-check',
                    additionalArguments: """
                    --scan bookmyshow-app
                    --exclude **/node_modules/**
                    --data ${ODC_DATA}
                    --noupdate
                    """
                )
            }
        }

        // ---------------- TRIVY SCAN ----------------
        stage('Trivy File Scan') {
            steps {
                sh '''
                trivy fs --severity HIGH,CRITICAL bookmyshow-app
                '''
            }
        }

        // ---------------- DOCKER BUILD & PUSH ----------------
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-CRED', toolName: 'Docker') {
                        sh '''
                        docker build -t mehardocker45/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app
                        docker push mehardocker45/bms:latest
                        '''
                    }
                }
            }
        }

        // ---------------- DEPLOY ----------------
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

    // ---------------- EMAIL ----------------
    post {
        always {
            emailext(
                to: 'prsam.789@gmail.com',
                subject: "Jenkins Build: ${currentBuild.currentResult}",
                body: """
                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                Status: ${currentBuild.currentResult}
                Build URL: ${env.BUILD_URL}
                """
            )
        }
    }
}
