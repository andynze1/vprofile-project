pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    environment{
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'please'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpr-maven-central'
        NEXUSIP = '10.1.1.96'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
    }
    stages{
        stage ('Build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post{
                success{
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        stage('Unit Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Code Analysis with Checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
            post{
                success {
                    echo 'Generated Analysis Result'
                    }
                }
            }
        }
    }
}