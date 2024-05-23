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
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }
    stages{
        stage('Build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests clean install'
            }
            post{
                success{
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Unit Test'){
            steps{
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Code Analysis with Checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
            post{
                success{
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('Code Analysis with Sonarqube') {
          
		  environment {
             scannerHome = tool 'sonarscanner'
          }

            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                    }

                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
            }
          }
        }
    }
}
