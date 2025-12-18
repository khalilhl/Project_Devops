pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    
    // Déclenchement automatique : vérification toutes les minutes pour détecter les nouveaux commits
    triggers {
        pollSCM('* * * * *') // Polling toutes les minutes (format cron: minute heure jour mois jour-semaine)
    }
    
    environment {
        // Configuration Docker Registry (à adapter selon votre registre)
        DOCKER_REGISTRY = 'docker.io' // ou 'registry.example.com' pour un registre privé
        DOCKER_IMAGE_NAME = 'khalilhlila/student-management'
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
        SONAR_TOKEN = credentials('sonar-token')
    }
    
    stages {
        stage('GIT - Récupération du code') {
            steps {
                script {
                    echo 'Récupération des dernières mises à jour du dépôt Git...'
                    git branch: 'master',
                        url: 'https://github.com/khalilhl/Project_Devops.git',
                        credentialsId: 'jenkins-github-credentials'
                    sh 'git log -1 --oneline'
                }
            }
        }
        
        stage('Build - Nettoyage et compilation') {
            steps {
                script {
                    echo 'Nettoyage et reconstruction du projet...'
                    sh 'mvn clean compile'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo 'Exécution des tests avec profil test (H2 en mémoire)...'
                    sh 'mvn test -Dspring.profiles.active=test'
                }
            }
            post {
                always {
                    // Publier les résultats des tests
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                sh """
                mvn sonar:sonar \
                -Dsonar.projectKey=my-project \
                -Dsonar.host.url=http://192.168.50.4:9000 \
                -Dsonar.login=$SONAR_TOKEN
                """
            }
        }
        
        stage('Package - Création du JAR') {
            steps {
                script {
                    echo 'Création du package JAR...'
                    sh 'mvn package -DskipTests'
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Docker - Build de l\'image') {
            steps {
                script {
                    echo "Construction de l'image Docker..."
                    sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                        docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }
        
        stage('Docker - Push vers le registre') {
            steps {
                script {
                    echo "Publication de l'image Docker dans le registre..."
                    try {
                        withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                                          usernameVariable: 'DOCKER_USER', 
                                                          passwordVariable: 'DOCKER_PASS')]) {
                            sh """
                                echo \$DOCKER_PASS | docker login ${DOCKER_REGISTRY} -u \$DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                                docker push ${DOCKER_IMAGE_NAME}:latest
                            """
                        }
                        echo "✅ Image Docker poussée avec succès vers le registre!"
                    } catch (Exception e) {
                        echo "⚠️  ATTENTION: Échec du push Docker vers le registre"
                        echo "⚠️  Raison: ${e.getMessage()}"
                        echo "⚠️  L'image Docker a été construite avec succès localement: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                        echo "⚠️  Pour résoudre ce problème:"
                        echo "   1. Créez le repository 'student-management' sur Docker Hub (https://hub.docker.com)"
                        echo "   2. Ou vérifiez que le nom d'utilisateur Docker Hub correspond à 'kacem-trabelsi'"
                        echo "   3. Ou vérifiez les permissions du repository"
                        // Ne pas faire échouer le pipeline si le push échoue
                        // Le build et l'image Docker sont créés avec succès
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
            echo "✅ Image Docker construite: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
            echo "✅ Image Docker construite: ${DOCKER_IMAGE_NAME}:latest"
        }
        failure {
            echo '❌ Pipeline échoué. Vérifiez les logs pour plus de détails.'
        }
        always {
            echo '🧹 Nettoyage des ressources...'
            // Optionnel: nettoyer les images Docker locales
            // sh 'docker image prune -f'
        }
    }
}