pipeline {
    agent any
    tools {
        jdk 'JDK17'
        maven 'MAVEN3.9'
    }

    environment {

        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }
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
         stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('SonarQube Analysis') {
    environment {
        scannerHome = tool "${SONARSCANNER}"
            steps {
                    withSonarQubeEnv("${SONARSCANNER}") {
                        sh '''${scnerHome}/bin/bash/sonar-scanner -Dsonar.project=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binares=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
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
