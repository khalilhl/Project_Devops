pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    environment {
        DOCKER_REGISTRY = 'docker.io' // Modifier selon votre registre (Docker Hub, GitLab Registry, etc.)
        DOCKER_IMAGE_NAME = 'khalilhl/student-management'
        DOCKER_CREDENTIALS_ID = 'docker-credentials' // ID des credentials Docker dans Jenkins
    }
    
    // Triggers commentés pour utiliser les webhooks GitHub avec NGROK
    // Décommentez la section ci-dessous si vous préférez utiliser le polling SCM
    /*
    triggers {
        // Vérifie les changements Git toutes les minutes
        pollSCM('* * * * *')
    }
    */
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis Git...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Nettoyage et reconstruction du projet...'
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    echo 'Build réussi! ✅'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    def latestTag = "${env.DOCKER_IMAGE_NAME}:latest"
                    
                    echo "Construction de l'image Docker: ${imageTag}"
                    sh "docker build -t ${imageTag} -t ${latestTag} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CREDENTIALS_ID}", 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${env.DOCKER_PASS} | docker login ${env.DOCKER_REGISTRY} -u ${env.DOCKER_USER} --password-stdin"
                        
                        def imageTag = "${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        def latestTag = "${env.DOCKER_IMAGE_NAME}:latest"
                        
                        echo "Publication de l'image Docker: ${imageTag}"
                        sh "docker push ${imageTag}"
                        sh "docker push ${latestTag}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline terminé.'
            cleanWs()
        }
        success {
            echo 'Pipeline réussi! ✅'
        }
        failure {
            echo 'Pipeline échoué! ❌'
        }
    }
}

