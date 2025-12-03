pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    stages {
        stage('GIT') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/khalilhl/Project_Devops.git',
                    credentialsId: 'jenkins-example-github-pat'
            }
        }
        
        stage('MAVEN') {
            steps {
                sh "mvn -version"
            }
        }
    }
}

