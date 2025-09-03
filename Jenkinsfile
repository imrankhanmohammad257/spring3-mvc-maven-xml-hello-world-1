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

        stage('Publish to Nexus') {
    steps {
        script {
            // Read Maven POM to get artifact coordinates
            def pom = readMavenPom file: 'pom.xml'
            def artifactVersion = pom.version
            def groupId = pom.groupId
            def artifactId = pom.artifactId

            // Find the WAR file in target folder
            def warFiles = findFiles(glob: "target/${artifactId}-${artifactVersion}.war")
            if (warFiles.length == 0) {
                error "WAR file not found: target/${artifactId}-${artifactVersion}.war"
            }
            def warFile = warFiles[0].path

            echo "Uploading artifact: ${warFile} (version: ${artifactVersion}) to Nexus"

            // Nexus upload
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
                credentialsId: 'nexus-creds',  // your Jenkins credentials ID for Nexus
                groupId: groupId,
                version: artifactVersion,
                repository: 'releases'  // Nexus repository name
            )
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
