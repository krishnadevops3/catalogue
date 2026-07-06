pipeline {

    agent {
        node {
            label 'AGENT-1'
        }
    }

    tools {
    nodejs 'NodeJS 20'
}

    environment {
        COURSE      = "Jenkins"
        ACC_ID      = "690509490317"
        PROJECT     = "roboshop"
        COMPONENT   = "catalogue"
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    env.appVersion = packageJSON.version

                    echo "===================================="
                    echo "Application Version : ${env.appVersion}"
                    echo "===================================="
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('Sonar Scan') {
            steps {
                script {

                    def scannerHome = tool 'sonar-8.0'

                    withSonarQubeEnv('sonar-server') {

                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${COMPONENT} \
                        -Dsonar.projectName=${COMPONENT} \
                        -Dsonar.sources=. \
                        -Dsonar.projectVersion=${env.appVersion}
                        """

                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {

                    withAWS(region: 'us-east-1', credentials: 'aws-creds') {

                        sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

                        docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion} .

                        docker images

                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion}
                        """

                    }

                }
            }
        }

        /*
        stage('Trivy Scan') {
            steps {
                sh """
                trivy image \
                --scanners vuln \
                --severity HIGH,CRITICAL,MEDIUM \
                --pkg-types os \
                --exit-code 1 \
                --format table \
                ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion}
                """
            }
        }
        */

    }

    post {

        always {
            echo "Cleaning Workspace..."
            cleanWs()
        }

        success {
            echo "Pipeline Executed Successfully."
        }

        failure {
            echo "Pipeline Execution Failed."
        }

        aborted {
            echo "Pipeline Aborted."
        }
    }
}