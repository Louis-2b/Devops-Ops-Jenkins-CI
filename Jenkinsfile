pipeline {
    // Spécifie l'agent Jenkins où s'exécutera le pipeline
    agent { label 'jenkins_agent' }  // nom du label de ton agent

    // Configure les outils nécessaires pour le pipeline
    tools {
        jdk "JDK17"  // Utilise JDK version 17
        maven "MAVEN3.9"  // Utilise Maven version 3.9
    }	
    
    // Définit les variables d'environnement pour tout le pipeline
    environment {
        SNAP_REPO = 'tubie-ops-snapshot'        // Dépôt Nexus pour les snapshots
        NEXUS_USER = 'admin'                    // Utilisateur Nexus
        NEXUS_PASS = credentials('nexus-creds') // Mot de passe Nexus (meilleure pratique: utiliser credentials)
        RELEASE_REPO = 'tubie-ops-release'      // Dépôt Nexus pour les releases
        CENTRAL_REPO = 'tubie-maven-central'    // Dépôt central Maven
        NEXUSIP = 'nexus.tubie.devops.ops'      // IP/hostname du serveur Nexus
        NEXUSPORT = '8081'                      // Port du serveur Nexus
        NEXUS_GRP_REPO = 'tubie-maven-group'    // Dépôt groupe Maven
        NEXUS_LOGIN = 'nexus_login'             // ID de connexion Nexus
    }
    
    // Définit les étapes du pipeline
    stages {
        // Étape 1: Compilation du code
        stage('Build') {
            steps {
            // Exécute Maven avec:
            // - settings.xml personnalisé
            // - Skip (Ignore) les tests (mais compilation maintenue)
             sh 'mvn -s settings.xml -DskipTests install'    
            }
        
            post {
                success {
                    echo "Build réussi - Archivage des artefacts..."
                    // Archive les fichiers .war générés
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        
        // Étape 2: Exécution des tests unitaires
        stage('Test') {
            steps {
                // Exécute les tests unitaires
                sh 'mvn test'
            }
        }
        
        // Étape 3: Analyse de qualité du code
        stage('Checkstyle Analysis') {
            steps {
                // Exécute l'analyse Checkstyle
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}

