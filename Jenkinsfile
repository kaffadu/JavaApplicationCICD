pipeline {
    agent any
    tools {
        maven 'maven386'  // Specific Maven version named 'maven386' in Jenkins Global Tool Configuration
    }
    
    environment {
        SONARQUBE_SERVER = 'SonarQubeServer'  // Name of the SonarQube server configured in Jenkins
        TOMCAT_USER = credentials('tomcat-user')  // Credentials ID for Tomcat authentication
        NEXUS_USER = credentials('nexus-user')  // Credentials ID for Nexus authentication
        NEXUS_URL_RELEASES = 'http://192.168.1.123:8081/repository/maven-releases/'  // Nexus releases repository URL
        NEXUS_URL_SNAPSHOTS = 'http://192.168.1.123:8081/repository/app-snapshot/'  // Nexus snapshots repository URL
        TOMCAT_URL = 'http://192.168.1.123:8082'  // URL for Tomcat manager
        APP_NAME = 'maven-web-application'
        COMPANY_NAME = 'com.mt'
        GIT_REPO_URL = 'https://github.com/kaffadu/JavaApplicationCICD.git'
        SONARQUBE_TOKEN = credentials('SonarQubeToken')  // Credentials ID for SonarQube token
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')  // DockerHub credentials
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
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh "mvn sonar:sonar -Dsonar.token=${SONARQUBE_TOKEN}"  // Use the token for authentication
                }
            }
        }

        stage('Upload to Nexus Releases') {
            steps {
                script {
                    def artifactVersion = "0.0.2-SNAPSHOT"  // Example version (adjust as needed)
                    def warFile = "target/${APP_NAME}-${artifactVersion}.war"  // Path to WAR file
                    
                    // Bind the credentials using Jenkins Credentials Plugin
                    withCredentials([usernamePassword(credentialsId: 'nexus-user', usernameVariable: 'NEXUS_USER_USR', passwordVariable: 'NEXUS_USER_PSW')]) {
                        // Use Maven to deploy the artifact to Nexus releases repository
                        sh """
                            mvn deploy:deploy-file \
                            -DgroupId=${COMPANY_NAME} \
                            -DartifactId=${APP_NAME} \
                            -Dversion=${artifactVersion} \
                            -Dpackaging=war \
                            -Dfile=${warFile} \
                            -DrepositoryId=nexus \
                            -Durl=${NEXUS_URL_RELEASES} \
                            -Drepository.username=${NEXUS_USER_USR} \
                            -Drepository.password=${NEXUS_USER_PSW} \
                            -Dmaven.metadata=false
                        """
                    }
                }
            }
        }

        stage('Upload to Nexus Snapshots') {
            steps {
                script {
                    def artifactVersion = "0.0.2-SNAPSHOT"  // Example version (adjust as needed)
                    def warFile = "target/${APP_NAME}-${artifactVersion}.war"  // Path to WAR file
                    
                    // Bind the credentials using Jenkins Credentials Plugin
                    withCredentials([usernamePassword(credentialsId: 'nexus-user', usernameVariable: 'NEXUS_USER_USR', passwordVariable: 'NEXUS_USER_PSW')]) {
                        // Use Maven to deploy the artifact to Nexus snapshots repository
                        sh """
                            mvn deploy:deploy-file \
                            -DgroupId=${COMPANY_NAME} \
                            -DartifactId=${APP_NAME} \
                            -Dversion=${artifactVersion} \
                            -Dpackaging=war \
                            -Dfile=${warFile} \
                            -DrepositoryId=nexus \
                            -Durl=${NEXUS_URL_SNAPSHOTS} \
                            -Drepository.username=${NEXUS_USER_USR} \
                            -Drepository.password=${NEXUS_USER_PSW} \
                            -Dmaven.metadata=false
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = "target/${APP_NAME}.war"  // Path to WAR file for deployment
                    
                    withCredentials([usernamePassword(credentialsId: 'tomcat-user', usernameVariable: 'TOMCAT_USER_USR', passwordVariable: 'TOMCAT_USER_PSW')]) {
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

        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "${DOCKERHUB_USERNAME}/${APP_NAME}:latest"
                    def dockerfilePath = 'Dockerfile'  // Adjust if your Dockerfile is in a different location
                    
                    sh """
                        docker build -t ${imageName} -f ${dockerfilePath} .
                    """
                }
            }
        }
        
        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        def imageName = "${DOCKERHUB_USERNAME}/${APP_NAME}:latest"
                        sh """
                            echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                            docker push ${imageName}
                        """
                    }
                }
            }
        }

    }         
}
