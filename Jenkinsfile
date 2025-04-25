pipeline {
    agent { label 'jenkins_agent' }  // nom du label de ton agent
    tools {
        jdk "JDK17"
        maven "MAVEN3.9"
    }	
    
    environmen {
        SNAP_REPO = 'tubie-ops-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'adminuser'
        RELEASE_REPO = 'tubie-ops-release'
        CENTRAL_REPO = 'tubie-maven-central'
        NEXUSIP = 'nexus.tubie.devops.ops'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'tubie-maven-group'
        NEXUS_LOGIN = 'nexus_login'
    }
    
    stages {
        stage('Build') {
            steps {
             // Exécute la phase 'install' de Maven avec un fichier settings personnalisé,
             // en ignorant l'exécution des tests (mais les tests sont quand meme compilés)
             sh 'mvn -s settings.xml -DskipTests install'    
            }
        }
    }
}

