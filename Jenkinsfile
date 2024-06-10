def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'UNSTABLE': 'warning',
    'ABORTED': '#808080'
]

pipeline {
    agent any
    
    environment {
        NEXUSPASS = credentials('nexuspass')
    }
    stages {
        stage('Setup Parameters'){
            steps {
                script {
                    properties([
                        parameters([
                            string(
                                defaultValue: '',
                                name: 'BUILD',
                            ),
                            string(
                            defaultValue:'',
                                name: 'TIME',
                            )
                        ])
                    ])
                }
            }
        }

        stage("Ansible Deploy to Prod") {
            steps {
                ansiblePlaybook([
                inventory   :   'ansible/prod.inventory',
                playbook    :   'ansible/site.yml',
                installation:   'ansible',
                colorized   :   true,
                credentialsId: 'applogin-prod',
                disableHostKeyChecking: true,
                extraVars   : [
                    USER: "admin",
                    PASS: "${NEXUSPASS}",
                    nexusip: "10.1.1.159",
                    reponame: "vprofile-release",
                    groupid: "QA",
                    time: "${env.TIME}",
                    build: "${env.BUILD}",
                    artifactid: "vproapp",
                    vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"
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
