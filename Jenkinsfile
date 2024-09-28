pipeline {
    agent any
    
    environment {
        MAVEN_HOME = tool(name: 'Maven368', type: 'maven')  // Use the installed Maven version
        SONARQUBE_SERVER = 'SonarQubeServer'  // Name of the SonarQube server configured in Jenkins
        NEXUS_USER = credentials('admin')  // Credentials ID for Nexus
        NEXUS_URL = 'http://192.168.1.123:8081//repository/maven-releases/'  // Nexus repository URL
        TOMCAT_URL = 'http://192.168.1.123:8082/manager/text'  // URL for Tomcat manager
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the repository
                git url: 'https://github.com/kaffadu/JavaApplicationCICD.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                // Clean and package the application using Maven
                sh "${MAVEN_HOME}/bin/mvn clean package"
            }
        }

        stage('Test') {
            steps {
                // Run tests using Maven
                sh "${MAVEN_HOME}/bin/mvn test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Perform SonarQube analysis
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh "${MAVEN_HOME}/bin/mvn sonar:sonar"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Wait for the SonarQube analysis to complete and check the quality gate status
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    def artifactVersion = "1.0.0"  // Update to the appropriate versioning
                    def warFile = "target/mywebapp-${artifactVersion}.war"
                    
                    // Use Maven deploy to send artifact to Nexus
                    sh """
                        ${MAVEN_HOME}/bin/mvn deploy:deploy-file \
                        -DgroupId=com.grantbase \
                        -DartifactId=mywebapp \
                        -Dversion=${artifactVersion} \
                        -Dpackaging=war \
                        -Dfile=${warFile} \
                        -DrepositoryId=nexus \
                        -Durl=${NEXUS_URL} \
                        -Drepository.username=${NEXUS_USER_USR} \
                        -Drepository.password=${NEXUS_USER_PSW}
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                // Deploy the war file to Tomcat
                script {
                    def warFile = "target/your-app-name.war"  // Path to the generated WAR file
                    sh """
                        --upload-file ${warFile} \
                        ${TOMCAT_URL}/deploy?path=/your-app&update=true
                    """
                }
            }
        }
    }

    post {
        always {
            // Archive the built artifacts, even if the build fails
            archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
        }
        success {
            // Notify on success
            echo 'Build and deployment successful!'
        }
        failure {
            // Notify on failure
            echo 'Build or deployment failed.'
        }
    }
}




