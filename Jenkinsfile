pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        NEXUS_REPO_URL = 'http://localhost:8081/repository/CICD/'
        REGISTRY = 'docker.io/shymaa145'
        IMAGE = 'react-app'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'dev',
                    url: 'https://github.com/shymaasaed/End-to-End-Tetris-Project.git'
            }
        }

        stage('Install & Build') {
            steps {
                sh '''
                node -v
                rm -rf node_modules
                npm install
                npm run build
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                npm test -- --ci --coverage || true
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('sonarqube') {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            script {
                                def scannerHome = tool 'sonar-scanner'

                                sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=CICD \
                                -Dsonar.projectName=CICD \
                                -Dsonar.sources=src \
                                -Dsonar.host.url=http://localhost:9000 \
                                -Dsonar.login=$SONAR_TOKEN
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Quality Gate (Non Blocking)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false
                    }
                }
            }
        }

        stage('OWASP Dependency Check (Non Blocking)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                    mkdir -p owasp-report dc-data

                    dependency-check.sh \
                    --scan . \
                    --format HTML \
                    --out owasp-report \
                    --data dc-data
                    '''
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS')]) {

                    sh '''
                    rm -rf deploy
                    mkdir deploy
                    cp -r build/* deploy/

                    cd deploy
                    zip -r CICD.zip .

                    curl -u $NEXUS_USER:$NEXUS_PASS --upload-file CICD.zip $NEXUS_REPO_URL
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Trivy Scan (Non Blocking)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                    trivy image --scanners vuln $REGISTRY/$IMAGE:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'shymaa145',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS')]) {

                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $REGISTRY/$IMAGE:$BUILD_NUMBER
                    '''
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'owasp-report/**', allowEmptyArchive: true
        }
    }
}
