pipeline {
    agent any
    tools {
        jdk 'JDK17'
        maven 'MAVEN3.9'
    }

    // environment {
    //     SNAP_REPO = 'vprofile-snapshot'
    //     NEXUS_USER = 'admin'
    //     NEXUS_PASS = 'admin123'
    //     RELEASE_REPO = 'vprofile-release'
    //     CENTRAL_REPO = 'vpro-maven-central'
    //     NEXUSIP = '172.31.94.8'
    //     NEXUSPORT = '8081'
    //     NEXUS_GRP_REPO = 'vpro-maven-group'
    //     NEXUS_LOGIN = 'nexuslogin'

    stages {
        
        stage('Build') {
            steps {
                sh 'mvn -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: 'target/*.war'
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
stage('SonarQube Analysis') {
            steps {
                // 'withSonarQubeEnv' sets up environment variables like SONAR_HOST_URL and SONAR_AUTH_TOKEN
                // based on the SonarQube server configuration named 'sonarserver' in Jenkins.
                script {
                    withSonarQubeEnv('sonarserver') { // 'sonarserver' is the name of your SonarQube server configuration in Jenkins
                        // Execute SonarQube analysis using Maven.
                        // -Dsonar.projectKey: This *must* be the unique key of your project in SonarQube.
                        // Replace 'your_vprofile_project_key' with the actual key you defined in SonarQube for this project.
                        // The sonar.host.url and sonar.login (for the token) are automatically provided by withSonarQubeEnv.
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar ' +
                           '-Dsonar.projectKey=your_vprofile_project_key ' + // IMPORTANT: Update this to your actual project key
                           '-Dsonar.host.url=$SONAR_HOST_URL ' +
                           '-Dsonar.login=$SONAR_AUTH_TOKEN'
                    }
                }
            }
            post {
                // Optional: Check SonarQube Quality Gate status and fail the build if it's not passed.
                // This requires the Jenkins SonarQube Quality Gates plugin and proper configuration.
                always {
                    script {
                        // Sleep for a few seconds to allow SonarQube to process the analysis results
                        // before checking the Quality Gate. Adjust the sleep duration if needed.
                        echo "Waiting for SonarQube analysis to complete and Quality Gate status..."
                        sleep 30 // Wait for 30 seconds (adjust as necessary)
                        def qg = waitForQualityGate() // Requires SonarQube Quality Gates plugin
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        } else {
                            echo "SonarQube Quality Gate passed: ${qg.status}"
                        }
                    }
                }
            }
        }
        

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}


// pipeline {
//     agent any
//     tools {
//         jdk 'JDK17'
//         maven 'MAVEN3.9'
//     }

//       // environment {
//       // SNAP_REPO = 'vprofile-snapshot'
//       // NEXUS_USER = 'admin'
//       // NEXUS_PASS = 'admin123'
//       // RELEASE_REPO = 'vprofile-release'
//       // CENTRAL_REPO = 'vpro-maven-central'
//       // NEXUSIP = '172.31.94.8'
//       // NEXUSPORT = '8081'
//       // NEXUS_GRP_REPO = 'vpro-maven-group'
//       // NEXUS_LOGIN = 'nexuslogin'
//       // }

//     stages {
        
//         stage('Build') {
//             steps {
//                 sh 'mvn -DskipTests install'
//             }
//         } 
//     }
// }
