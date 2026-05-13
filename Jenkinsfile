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

stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube-Scanner') {
            sh '''
            sonar-scanner \
            -Dsonar.projectName=BMS \
            -Dsonar.projectKey=BMS \
            -Dsonar.sources=. \
            -Dsonar.tests=.
            '''
        }
    }
}
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false,
                credentialsId: 'Sonar-token'
            }
        }

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

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments:
                '--scan ./ --disableYarnAudit --disableNodeAudit',
                odcInstallation: 'DP-Check'

                dependencyCheckPublisher pattern:
                '**/dependency-check-report.xml'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

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

                echo "Running containers:"
                docker ps -a

                echo "Waiting for application startup..."
                sleep 10

                docker logs bms
                '''
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                Project: ${env.JOB_NAME}<br/>
                Build Number: ${env.BUILD_NUMBER}<br/>
                URL: ${env.BUILD_URL}<br/>
                """,
                to: 'msan8795@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
        }
    }
}
