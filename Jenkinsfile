pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'MAVEN3.9'
    }

    environment {
        // Define your SonarQube server name as configured in Jenkins > Manage Jenkins > Configure System > SonarQube servers
        SONARQUBE_SERVER_NAME = 'sonarserver'  // Replace with actual configured name
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER_NAME}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage("Quality Gate") {
            steps {
                // Optional sleep to give SonarQube time to analyze
                sleep time: 60, unit: 'SECONDS'
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
