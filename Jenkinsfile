pipeline {
    agent any
    tools {
        maven 'maven386'  // Specific Maven version named 'maven386' in Jenkins Global Tool Configuration
    }
    
    environment {
        // SONARQUBE_SERVER = 'SonarQubeServer'  // Name of the SonarQube server configured in Jenkins
        TOMCAT_USER = credentials('tomcat-user')  // Credentials ID for Tomcat authentication
        NEXUS_USER = credentials('nexus-user')  // Credentials ID for Nexus authentication
        NEXUS_URL = 'http://192.168.1.123:8081/repository/maven-releases/'  // Nexus repository URL
        TOMCAT_URL = 'http://192.168.1.123:8082/manager/text'  // URL for Tomcat manager
        APP_NAME = 'mywebapp'
        COMPANY_NAME = 'grantbase'
        GIT_REPO_URL = 'https://github.com/kaffadu/JavaApplicationCICD.git'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from the Git repository
                git url: "${GIT_REPO_URL}", branch: 'master'
            }
        }

        stage('Build') {
            steps {
                // Clean and package the application using Maven
                sh "mvn clean package"
            }
        }

        stage('Test') {
            steps {
                // Run tests using Maven
                sh "mvn test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Perform SonarQube analysis
                withSonarQubeEnv('SonarQubeServer') {
                    sh "mvn sonar:sonar"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Wait for SonarQube quality gate results
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    def artifactVersion = "1.0.0"  // Example version (adjust as needed)
                    def warFile = "target/${APP_NAME}-${artifactVersion}.war"  // Path to WAR file
                    
                    // Use Maven to deploy the artifact to Nexus repository
                    sh """
                        mvn deploy:deploy-file \
                        -DgroupId=${COMPANY_NAME} \
                        -DartifactId=${APP_NAME} \
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
                script {
                    def warFile = "target/${APP_NAME}.war"  // Path to WAR file for deployment

                    // Accessing the credentials directly
                    def tomcatUser = TOMCAT_USER_USR
                    def tomcatPassword = TOMCAT_USER_PSW
                    
                    // Deploy the WAR file to Tomcat using curl
                    sh """
                        curl --user ${TOMCAT_USER_USR}:${TOMCAT_USER_PSW} \
                        --upload-file ${warFile} \
                        ${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true
                    """
                }
            }
        }
    }
}
