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
        // Configuration Nexus
        SNAP_REPO = 'tubie-ops-snapshot'        // Dépôt Nexus pour les snapshots
        NEXUS_USER = 'admin'                    // Utilisateur Nexus
        NEXUS_PASS = 'adminuser'                // Mot de passe Nexus
        RELEASE_REPO = 'tubie-ops-release'      // Dépôt Nexus pour les releases
        CENTRAL_REPO = 'tubie-maven-central'    // Dépôt central Maven
        NEXUSIP = 'nexus.tubie.devops.ops'      // IP/hostname du serveur Nexus
        NEXUSPORT = '8081'                      // Port du serveur Nexus
        NEXUS_GRP_REPO = 'tubie-maven-group'    // Dépôt groupe Maven
        NEXUS_LOGIN = 'nexus_login'             // ID de connexion Nexus
        
        // Configuration SonarQube
        SONARSERVER = 'sonarserver'             // Nom du serveur Sonar configuré dans Jenkins
        SONARSCANNER = 'sonarscanner'           // Nom du scanner Sonar configuré dans Jenkins
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
                sh 'mvn -s settings.xml test'
            }
        }
        
        // Étape 3: Analyse de qualité du code
        stage('Checkstyle Analysis') {
            steps {
                // Exécute l'analyse Checkstyle
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        // Étape 4: SonarQube - Analyse statique approfondie
        stage('SonarQube analysis') {
            environment {
                /*
                 * Définit le chemin d'accès au scanner Sonar
                 * ${SONARSCANNER} doit correspondre à un outil configuré dans Jenkins
                 */
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    /*
                     * Exécute le scanner Sonar avec les paramètres:
                     * - projectKey: Identifiant unique du projet dans Sonar
                     * - projectName: Nom affiché dans Sonar
                     * - sources: Répertoire des sources à analyser
                     * - java.binaries: Répertoire des classes compilées
                     * - junit.reportsPath: Emplacement des rapports de test
                     * - jacoco.reportsPath: Emplacement du rapport de couverture
                     * - checkstyle.reportPaths: Emplacement du rapport Checkstyle
                     */
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=tubie-ops-java \
                    -Dsonar.projectName=tubie-ops-java \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
    }
}

