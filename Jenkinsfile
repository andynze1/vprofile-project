def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'UNSTABLE': 'warning',
    'ABORTED': '#808080'
]

pipeline {
    agent any
    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK8'
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'please'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpr-maven-central'
        NEXUSIP = '10.1.1.101'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        NEXUSPASS = credentials('nexuspass')
    //    NEXUS_PROTOCOL = 'http'

//        NEXUS_REPOGRP_ID = 'QA'
//        NEXUS_VERSION = 'nexus3'
    }
    stages {
        stage('Build Artifact') {
            steps {
                sh 'mvn -s settings.xml -DskipTests clean install'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Unit Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Code Analysis with Checkstyle') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('Code Analysis with SonarQube') {
            environment {
                scannerHome = tool SONARSCANNER
            }
            steps {
                withSonarQubeEnv(SONARSERVER) {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Upload Artifacts to Nexus Repository Manager') {
            steps {
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}",
                credentialsId: "${NEXUS_LOGIN}",
                artifacts: [
                    [artifactId: 'vproapp',
                    classifier: '',
                    file: 'target/vprofile-v2.war',
                    type: 'war']
                ]
                )
            }
        }

        stage("Ansible Deploy to Staging") {
            steps {
                ansiblePlaybook([
                inventory   :   'ansible/stage.inventory',
                playbook    :   'ansible/site.yml',
                installation:   'ansible',
                colorized   :   true,
                credentialsId: 'applogin',
                disableHostKeyChecking: true,
                extraVars   : [
                    USER: "admin",
                    PASS: "${NEXUSPASS}",
                    nexusip: "10.1.1.101",
                    reponame: "vprofile-release",
                    groupid: "QA",
                    time: "${env.BUILD_TIMESTAMP}",
                    build: "${env.BUILD_ID}",
                    artifactid: "vproapp",
                    vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
                ]
              ])
            }
        }

        stage("Debug") {
            steps {
                echo "Build ID: ${env.BUILD_ID}"
            }
        }
   }

    post {
        always {
            script {
               // Debug statements
               echo "NEXUS_USER: ${NEXUS_USER}"
               echo "NEXUS_PASS: ${NEXUS_PASS}"
               echo "NEXUSIP: ${NEXUSIP}"
               echo "NEXUSPORT: ${NEXUSPORT}"
               echo "RELEASE_REPO: ${RELEASE_REPO}"
               echo "BUILD_ID: ${env.BUILD_ID}"
               echo "BUILD_TIMESTAMP: ${env.BUILD_TIMESTAMP}"

                               // Slack notification
                def color = COLOR_MAP.get(currentBuild.currentResult, '#808080') // Default to gray if result not in map
                echo 'Slack Notification.'
                slackSend (
                    channel: '#jenkinscicd',
                    color: color,
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \nMore info at: ${env.BUILD_URL}"
                )
            }
        }
    }
}
