/*
 * [1] DÉCLARATION DU PIPELINE
 * Définit un pipeline Jenkins qui s'exécutera sur un agent spécifique
 */
pipeline {
    /*
     * [2] CONFIGURATION DE L'AGENT
     * Spécifie que le pipeline s'exécutera sur un nœud Jenkins avec le label 'jenkins_agent'
     */
    agent { label 'jenkins_agent' }

    /*
     * [3] CONFIGURATION DES OUTILS
     * Définit les outils nécessaires qui doivent être préconfigurés dans Jenkins
     */
    tools {
        jdk "JDK17"  // Requiert JDK 17 configuré dans "Manage Jenkins > Global Tool Configuration"
        maven "MAVEN3.9"  // Requiert Maven 3.9 configuré de la même manière
    }	
    
    /*
     * [4] VARIABLES D'ENVIRONNEMENT
     * Variables disponibles dans tout le pipeline
     */
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
    
    /*
     * [5] ÉTAPES DU PIPELINE
     * Contient toutes les étapes d'exécution séquentielles
     */
    stages {
        /*
         * [6] ÉTAPE BUILD - COMPILATION
         * Compile le code source et génère les artefacts
         */
        stage('Build') {
            steps {
            /*
             * Commande Maven:
             * - -s settings.xml : utilise un fichier de configuration Maven personnalisé
             * - -DskipTests : compile mais n'exécute pas les tests
             * - install : installe l'artefact dans le repository local
             */
             sh 'mvn -s settings.xml -DskipTests install'    
            }
            
            /*
             * [7] POST-ACTIONS DU BUILD
             * Actions exécutées après l'étape de build selon son statut
             */
            post {
                success {
                    echo "Build réussi - Archivage des artefacts..."
                    // Archive tous les fichiers .war trouvés dans l'espace de travail
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        
        /*
         * [8] ÉTAPE TEST - TESTS UNITAIRES
         * Exécute les tests unitaires et génère des rapports
         */
        stage('Test') {
            steps {
                /*
                 * Commande Maven:
                 * - test : exécute les tests unitaires
                 * - -s settings.xml : utilise la configuration personnalisée
                 * Génère des rapports dans target/surefire-reports/
                 */
                sh 'mvn -s settings.xml test'
            }
        }
        
        /*
         * [9] ÉTAPE CHECKSTYLE - ANALYSE DE CODE
         * Vérifie la conformité du code aux standards
         */
        stage('Checkstyle Analysis') {
            steps {
                /*
                 * Commande Maven:
                 * - checkstyle:checkstyle : exécute l'analyse Checkstyle
                 * Génère un rapport dans target/checkstyle-result.xml
                 */
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        /*
         * [10] ÉTAPE SONARQUBE - ANALYSE STATIQUE AVANCÉE
         * Effectue une analyse approfondie de la qualité du code
         */
        stage('SonarQube analysis') {
            environment {
                /*
                 * Définit le chemin d'accès au scanner Sonar
                 * ${SONARSCANNER} doit correspondre à un outil configuré dans Jenkins
                 */
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                /*
                 * Configure l'environnement SonarQube avec les credentials
                 * "${SONARSERVER}" doit correspondre à une configuration serveur dans Jenkins
                 */
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

        /*
         * [11] ÉTAPE QUALITY GATE - VALIDATION DE LA QUALITÉ
         * Attend et vérifie les résultats du Quality Gate de SonarQube
         */
        stage("Quality Gate") {
            steps {
                /*
                 * Définit un timeout de 1 heure pour éviter des attentes infinies
                 * waitForQualityGate vérifie le statut de l'analyse SonarQube:
                 * - abortPipeline: true => Arrête le pipeline si échec au Quality Gate
                 */
                timeout(time: 1, unit: 'HOURS') {
                    // Le paramètre indique s'il faut définir le pipeline sur INSTABLE en cas d'échec de Quality Gate
                    // true = définir le pipeline sur INSTABLE, false = ne pas le faire
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /*
         * [12] ÉTAPE UPLOAD ARTIFACT - DÉPLOIEMENT VERS NEXUS
         * Téléverse l'artefact généré vers le repository Nexus
         */
        stage("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                  // Configuration de base de Nexus
                  nexusVersion: 'nexus3',                                    // Version de Nexus (2 ou 3)
                  protocol: 'http',                                          // Protocole (http/https)
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",                       // URL de Nexus (variables d'environnement)
                  
                  // Métadonnées de l'artefact
                  groupId: 'QA',                                             // Groupe Maven (organisation)
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",         // Version unique basée sur l'ID et timestamp du build
                  repository: "${RELEASE_REPO}",                             // Dépôt cible (variable d'environnement)
                  credentialsId: "${NEXUS_LOGIN}",                           // Identifiant des credentials stockés dans Jenkins
                  
                  /*
                   * Liste des artefacts à uploader
                   * Ici nous uploadons un seul fichier .war
                   */
                  artifacts: [
                    [artifactId: 'tubie-ops-app',                            // ID de l'artefact
                     classifier: '',                                         // Classifieur (optionnel)
                     file: 'target/vprofile-v2.war',                         // Chemin du fichier à uploader
                     type: 'war']                                            // Type d'artefact (extension)
                  ]
                )
            }
        }
    }
}

