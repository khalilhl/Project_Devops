pipeline {
    agent any

    tools {
        maven 'M2_HOME'      // Ton installation Maven dans Jenkins
        jdk 'JAVA_HOME'       // Ton JDK installé dans Jenkins
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_REGISTRY = 'docker.io'   // Docker Hub
        DOCKER_IMAGE_NAME = 'khalilhlila/student-management'
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    triggers {
        pollSCM('* * * * *')  // Vérifie les commits toutes les minutes
    }

    stages {

        stage('GIT - Récupération du code') {
            steps {
                script {
                    echo 'Récupération du code depuis GitHub...'
                    git branch: 'master',
                        url: 'https://github.com/khalilhl/Project_Devops.git',
                        credentialsId: 'jenkins-github-credentials'
                    sh 'git log -1 --oneline'
                }
            }
        }

        stage('Build - Compilation') {
            steps {
                script {
                    echo '⚙️ Compilation du projet Maven...'
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Tests') {
            steps {
                script {
                    echo '🧪 Exécution des tests avec H2 (profil test)...'
                    sh 'mvn test -Dspring.profiles.active=test'
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo '🔍 Analyse SonarQube...'
                    sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=my-project \
                        -Dsonar.host.url=http://192.168.50.4:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Package - Création du JAR') {
            steps {
                script {
                    echo '📦 Création du JAR...'
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
                    echo "🐳 Construction de l'image Docker..."
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
                    echo "📤 Push de l'image Docker vers Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials',
                                                      usernameVariable: 'DOCKER_USER',
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login ${DOCKER_REGISTRY} -u \$DOCKER_USER --password-stdin
                            docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            docker push ${DOCKER_IMAGE_NAME}:latest
                        """
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
            echo '🧹 Nettoyage du workspace...'
            cleanWs()
        }
    }
}
