pipeline {
    agent any

    tools {
        maven "Maven-3.8.4"
    }

    environment {
        NEXUS_URL            = "54.163.17.174:8081"
        NEXUS_REPOSITORY     = "releases"
        NEXUS_CREDENTIAL_ID  = "nexus-creds"

        TOMCAT_USER          = "deployer"       // Tomcat manager username
        TOMCAT_PASSWORD      = "deployer"   // Tomcat manager password
        TOMCAT_HOST          = "3.82.42.125"       // Tomcat EC2 public IP
        TOMCAT_PORT          = "8080"
        SLACK_CHANNEL        = "#jenkins-integration"
        SLACK_CREDENTIAL_ID  = "slack_notification"
    }

    stages {
        stage("Clone code") {
            steps {
                git 'https://github.com/imrankhanmohammad257/spring3-mvc-maven-xml-hello-world-1.git'
            }
        }

        stage("Maven build") {
            steps {
                sh 'mvn -B -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactVersion = pom.version
                    def groupId = pom.groupId
                    def artifactId = pom.artifactId

                    def warFiles = findFiles(glob: "target/${artifactId}-${artifactVersion}.war")
                    if (warFiles.length == 0) {
                        error "WAR file not found: target/${artifactId}-${artifactVersion}.war"
                    }
                    def warFile = warFiles[0].path

                    echo "Uploading artifact: ${warFile} (version: ${artifactVersion}) to Nexus"

                    nexusArtifactUploader(
                        artifacts: [[
                            artifactId: artifactId,
                            classifier: '',
                            file: warFile,
                            type: 'war'
                        ], [
                            artifactId: artifactId,
                            classifier: '',
                            file: 'pom.xml',
                            type: 'pom'
                        ]],
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        groupId: groupId,
                        version: artifactVersion,
                        repository: NEXUS_REPOSITORY
                    )
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactVersion = pom.version
                    def artifactId = pom.artifactId
                    def warFile = "target/${artifactId}-${artifactVersion}.war"

                    echo "Deploying ${warFile} to Tomcat at ${TOMCAT_HOST}:${TOMCAT_PORT}"

                    sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} \
                         -T ${warFile} \
                         "http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/${artifactId}&update=true"
                    """
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: SLACK_CHANNEL,
                color: 'good',
                message: "✅ Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]' completed successfully! By IMRAN KHAN <${env.BUILD_URL}|Open Build>"
            )
            cleanWs()
        }
        failure {
            slackSend(
                channel: SLACK_CHANNEL,
                color: 'danger',
                message: "❌ Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed! <${env.BUILD_URL}|Open Build>"
            )
            cleanWs()
        }
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
