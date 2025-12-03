pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JDK17'
    }
    
    environment {
        DOCKER_REGISTRY = 'docker.io' // Modifier selon votre registre (docker.io pour Docker Hub)
        IMAGE_NAME = 'khalilhl/student-management'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    // Pas de section triggers nécessaire pour les webhooks GitHub
    // Les webhooks sont configurés côté GitHub et dans la configuration du job Jenkins
    // (Build Triggers -> GitHub hook trigger for GITScm polling)
    
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
                    def imageTag = "${IMAGE_NAME}:${IMAGE_TAG}"
                    def latestTag = "${IMAGE_NAME}:latest"
                    
                    echo "Construction de l'image Docker: ${imageTag}"
                    sh "docker build -t ${imageTag} -t ${latestTag} ."
                }
            }
        }
        
        stage('Push Image Docker') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} --password-stdin"
                        
                        def imageTag = "${IMAGE_NAME}:${IMAGE_TAG}"
                        def latestTag = "${IMAGE_NAME}:latest"
                        
                        echo "Publication de l'image Docker dans le registre..."
                        sh "docker push ${imageTag}"
                        sh "docker push ${latestTag}"
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