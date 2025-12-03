pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    environment {
        DOCKER_REGISTRY = 'your-registry-url' // Exemple: docker.io/username ou registry.example.com
        IMAGE_NAME = 'student-management'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    triggers {
        // Polling SCM toutes les minutes pour détecter les nouveaux commits
        // Plus fiable avec NGROK gratuit (évite la page d'avertissement qui bloque les webhooks)
        pollSCM('* * * * *')
    }
    
    stages {
        stage('GIT - Récupération du code') {
            steps {
                script {
                    echo 'Récupération des dernières mises à jour du dépôt Git...'
                }
                git branch: 'master',
                    url: 'https://github.com/khalilhl/Project_Devops.git',
                    credentialsId: 'jenkins-github-credentials'
            }
        }
        
        stage('Build Maven - Nettoyage et Construction') {
            steps {
                script {
                    echo 'Nettoyage et reconstruction du projet Maven avec tests...'
                }
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Build Maven réussi!'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
                failure {
                    echo 'Échec du build Maven!'
                }
                always {
                    // Publication des rapports de tests
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build Image Docker') {
            steps {
                script {
                    echo "Construction de l'image Docker: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Image Docker') {
            steps {
                script {
                    echo "Publication de l'image Docker dans le registre..."
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                        // Tag également comme 'latest'
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline exécuté avec succès!'
        }
        failure {
            echo 'Pipeline échoué!'
        }
        always {
            cleanWs()
        }
    }
}