pipeline {
    agent any

    tools {
        maven "Maven-3.8.4"
    }

    environment {
        NEXUS_VERSION        = "nexus3"
        NEXUS_PROTOCOL       = "http"
        NEXUS_URL            = "54.163.17.174:8081"
        NEXUS_REPOSITORY     = "releases"
        NEXUS_CREDENTIAL_ID  = "nexus-creds"

        // Tomcat deployment variables
        TOMCAT_USER          = "ec2-user"                // SSH user for Tomcat EC2
        TOMCAT_HOST          = "3.82.42.125"            // Tomcat EC2 IP
        TOMCAT_PATH          = "/opt/tomcat/webapps"
        TOMCAT_SSH_KEY       = "tomcat-ssh-key"         // Jenkins credentials ID for SSH private key
    }

    stages {
        stage("Clone code") {
            steps {
                git 'https://github.com/imrankhanmohammad257/spring3-mvc-maven-xml-hello-world-1.git'
            }
        }

        stage("Maven build") {
            steps {
                sh 'mvn -B -Dmaven.test.failure.ignore=true install'
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
                        credentialsId: 'nexus-creds',
                        groupId: groupId,
                        version: artifactVersion,
                        repository: 'releases'
                    )
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactVersion = pom.version
                    def artifactId = pom.artifactId
                    def warFile = "target/${artifactId}-${artifactVersion}.war"

                    echo "Deploying ${warFile} to Tomcat at ${TOMCAT_HOST}"

                    sshagent([TOMCAT_SSH_KEY]) {
                        // Stop Tomcat (optional)
                        sh "ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} 'sudo systemctl stop tomcat || true'"

                        // Copy WAR
                        sh "scp -o StrictHostKeyChecking=no ${warFile} ${TOMCAT_USER}@${TOMCAT_HOST}:${TOMCAT_PATH}/"

                        // Remove old exploded folder
                        sh "ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} 'rm -rf ${TOMCAT_PATH}/${artifactId}'"

                        // Start Tomcat
                        sh "ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} 'sudo systemctl start tomcat'"
                    }

                    echo "Deployment completed!"
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#jenkins-integration',
                color: 'good',
                message: "✅ Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]' completed successfully! <${env.BUILD_URL}|Open Build>"
            )
            cleanWs()
        }
        failure {
            slackSend(
                channel: '#jenkins-integration',
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
