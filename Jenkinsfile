pipeline {
    agent any

    tools {
        // Make sure this name matches exactly what you set in Manage Jenkins -> Global Tool Configuration
        maven "Maven-3.8.4"
    }

    environment {
        NEXUS_VERSION        = "nexus3"
        NEXUS_PROTOCOL       = "http"
        NEXUS_URL            = "54.163.17.174:8081"   // host:port (no protocol)
        NEXUS_REPOSITORY     = "devops"
        NEXUS_CREDENTIAL_ID  = "nexus-creds"        // must exist in Jenkins credentials
    }

    stages {
        stage("Clone code") {
            steps {
                git 'https://github.com/imrankhanmohammad257/spring3-mvc-maven-xml-hello-world-1.git'
            }
        }

        stage("Maven build") {
            steps {
                // use mvn from the tools block; "-B" for batch mode
                sh 'mvn -B -Dmaven.test.failure.ignore=true install'
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    // read the POM
                    def pom = readMavenPom file: 'pom.xml'
                    def packaging = pom.packaging ?: 'jar'          // fallback if packaging is null
                    def filesByGlob = findFiles(glob: "target/*.${packaging}")

                    if (filesByGlob.length == 0) {
                        error("No artifact found in target/*.${packaging}")
                    }

                    def artifact = filesByGlob[0]
                    echo "Found artifact: ${artifact.path} (size=${artifact.length}). groupId=${pom.groupId}, artifactId=${pom.artifactId}, pomVersion=${pom.version}"

                    // choose version to upload: recommended to combine pom.version + build number
                    def versionToUse = pom.version ? "${pom.version}-${env.BUILD_NUMBER}" : "${env.BUILD_NUMBER}"
                    echo "Uploading version: ${versionToUse} to Nexus repo ${NEXUS_REPOSITORY}"

                    nexusArtifactUploader(
                        nexusVersion: "${NEXUS_VERSION}",
                        protocol: "${NEXUS_PROTOCOL}",
                        nexusUrl: "${NEXUS_URL}",
                        groupId: pom.groupId,
                        version: "${versionToUse}",
                        repository: "${NEXUS_REPOSITORY}",
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: pom.artifactId, classifier: '', file: artifact.path, type: packaging],
                            [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                        ]
                    )

                    // archive (optional) so the artifact is saved with the build
                    archiveArtifacts artifacts: artifact.path, fingerprint: true
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
        always {
            cleanWs()
        }
    }
}
