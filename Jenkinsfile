pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'MAVEN3.9'
    }

    environment {
        NEXUS_USER     = 'admin'
        NEXUS_PASS     = 'admin123'
        NEXUSIP        = '172.31.94.8'
        NEXUSPORT      = '8081'
        RELEASE_REPO   = 'vprofile-release'
        NEXUS_LOGIN    = 'nexuslogin'         // ID of Jenkins credentials (username+password or token)
        SONARQUBE_SERVER_NAME = 'sonarqube-9' // Must match Jenkins > Manage Jenkins > Configure System
    }

    stages {
        stage('Build the code') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Verify WAR File') {
            steps {
                sh 'ls -lh target/'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER_NAME}") {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            mvn sonar:sonar \
                              -Dsonar.projectKey=vprofile \
                              -Dsonar.projectName=vprofile \
                              -Dsonar.host.url=http://44.202.20.235:9000 \
                              -Dsonar.login=$SONAR_TOKEN
                        '''
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

        stage('Upload Artifact to Nexus') {
            when {
                expression { fileExists('target/vprofile-v2.war') }
            }
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "http://${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [[
                        artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war'
                    ]]
                )
            }
        }
    }

    post {
        failure {
            echo 'Build failed. Check logs for details.'
        }
        success {
            echo 'Build, Sonar scan, and deployment successful!'
        }
    }
}
