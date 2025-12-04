pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    environment {
        DOCKER_REGISTRY = 'docker.io' // Modifier selon votre registre (docker.io pour Docker Hub)
        IMAGE_NAME = 'aminetriki/student-management'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        PUSH_DOCKER = 'true' // Mettre à 'true' pour activer le push Docker (nécessite credentials)
    }
    
    triggers {
        // Vérifie les changements Git toutes les minutes
        // Le pipeline se déclenchera automatiquement à chaque nouveau commit
         // Le pipeline se déclenchera automatiquement à chaque nouveau commit
        pollSCM('* * * * *')
    }
    
    stages {
        stage('GIT - Récupération du code') {
            steps {
                script {
                    echo 'Récupération des dernières mises à jour du dépôt Git...'
                }
                // Utilise checkout scm car le pipeline est configuré avec "Pipeline script from SCM"
                checkout scm
            }
        }
        
        stage('Build Maven - Nettoyage et Construction') {
            steps {
                script {
                    echo 'Nettoyage et reconstruction du projet Maven...'
                    // Skip tests car MySQL n'est pas disponible dans l'environnement Jenkins
                    sh 'mvn clean package -DskipTests'
                }
            }
            post {
                success {
                    echo 'Build Maven réussi!'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
                failure {
                    echo 'Échec du build Maven!'
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
            when {
                expression { env.PUSH_DOCKER == 'true' }
            }
            steps {
                script {
                    echo "Publication de l'image Docker dans le registre..."
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} --password-stdin"
                        
                        def imageTag = "${IMAGE_NAME}:${IMAGE_TAG}"
                        def latestTag = "${IMAGE_NAME}:latest"
                        
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
            echo 'Pipeline échouée!'
        }
        always {
            cleanWs()
        }
    }
}