// Define the URL of the Artifactory registry
def registry ='https://trialangzfu.jfrog.io/'

pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {

        stage("build") {
            steps {
                echo "----------- build started ----------"
                sh 'mvn clean package -DskipTests'
                echo "----------- build completed ----------"
            }
        }

        stage("test") {
            steps {
                echo "----------- unit test started ----------"
                sh 'mvn test'
                echo "----------- unit test completed ----------"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'SQ-Tool'
            }

            steps {
                withSonarQubeEnv('SQ-Server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'

                    def server = Artifactory.newServer(
                        url: registry + "/artifactory",
                        credentialsId: "/****** (JfrogToken)"
                    )

                    def properties = "buildid=${env.BUILD_ID}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "manidemy-libs-release-local/",
                                "flat": "true",
                                "props": "${properties}"
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)
                    server.publishBuildInfo(buildInfo)

                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
    }
}
