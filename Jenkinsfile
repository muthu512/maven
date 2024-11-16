pipeline {
    agent any

    environment {
        // Define project directory and other environment variables
        PROJECT_DIR = "C:\\Users\\Dell-Lap\\Downloads\\simple-springboot-app-master\\simple-springboot-app-master"
        DEPLOY_DIR = "C:\\Users\\Dell-Lap\\Downloads\\Newfolder" // The folder where the JAR will be deployed
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the source code from GitHub
                    checkout scm
                    // Optional: Log the latest commit hash for verification
                    bat 'git log -1'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Clean and build the project
                    bat "mvn clean package -f C:\\Users\\Dell-Lap\\Downloads\\simple-springboot-app-master\\simple-springboot-app-master\\pom.xml"

                    
                    // Log the build directory content for debugging
                    bat "dir ${PROJECT_DIR}\\target"
                }
            }
        }

        stage('Determine JAR Name') {
            steps {
                script {
                    // Find the JAR file in the target directory
                    def jarOutput = bat(
                        script: "for %i in (${PROJECT_DIR}\\target\\*.jar) do @echo %~nxi",
                        returnStdout: true
                    ).trim()

                    // Extract the JAR name and define it as an environment variable
                    def jarFiles = jarOutput.split("\n")
                    def jarFile = jarFiles.find { it.endsWith('.jar') && !it.contains('.original') }
                    if (!jarFile) {
                        error "No JAR file found in target directory!"
                    }
                    env.BUILD_JAR = "${PROJECT_DIR}\\target\\${jarFile}"
                    echo "Discovered JAR file: ${env.BUILD_JAR}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Define the deployment path for the JAR
                    def deployJar = "${DEPLOY_DIR}\\${env.BUILD_JAR.split('\\\\').last()}" // Extract the JAR name from the full path

                    // Verify if the built JAR exists
                    bat """
                    if not exist "${env.BUILD_JAR}" (
                        echo Build artifact not found! && exit 1
                    )
                    """

                    // Delete old JAR file in the deployment folder if it exists
                    bat """
                    if exist "${deployJar}" (
                        del /F "${deployJar}"
                    )
                    """

                    // Copy the new JAR to the deployment folder
                    bat "copy /Y \"${env.BUILD_JAR}\" \"${deployJar}\""

                    // Run the new JAR using Java with specified arguments
                    bat "start java -jar \"${deployJar}\" --spring.profiles.active=prod --server.port=1010"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
