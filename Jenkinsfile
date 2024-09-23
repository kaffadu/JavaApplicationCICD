pipeline {
    agent any
    tools{
    maven "maven386"
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub repository
                checkout([$class: 'GitSCM',
                        branches: [[name: '*/master']],
                        userRemoteConfigs: [[url: 'https://github.com/kaffadu/skillsedgelab.git']]])
            }
        }

        stage('2build') {
            steps{
                sh "mvn clean package"
                common("Build")
            }
        }


        stage('3CodeQualityAnalysis') {
            steps{
                sh "mvn sonar:sonar"
            }
        }
    }  
}
