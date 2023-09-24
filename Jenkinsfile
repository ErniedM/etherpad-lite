pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Haal de broncode op uit de Git-repository
                checkout scm
            }
        }

        stage('OWASP Dependency Check') {
            steps {
              dependencyCheck additionalArguments: '--scan /home/s127280/Opdracht1/etherpad-lite/'' 
                          -o './'
                          -s './'
                          -f 'ALL' 
                          --prettyPrint''', odcInstallation: 'owasp'
        
              dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

        stage('Build') {
            steps {
                // Voer de buildstappen uit (bijv. installatie van afhankelijkheden, bouwen van Etherpad)
                sh 'docker compose build'
            }
        }

        stage('Static Analysis') {
            steps {
                // Voer statische code-analyse uit (bijv. ESLint)
                sh 'echo "Running static analysis..."'
            }
        }

        stage('Security Scanning') {
            steps {
                // Voer beveiligingsscans uit (bijv. OWASP Dependency-Check, container scanning)
                sh 'echo "Running security scans..."'
            }
        }

        stage('Testing') {
            steps {
                // Voer tests uit (bijv. unit tests, functionele tests)
                sh 'echo "Running tests..."'
            }
        }

        stage('Deployment') {
            steps {
                // Implementeer Etherpad in de gewenste omgeving (bijv. productie)
                //sh 'echo "Deploying Etherpad..."'
                sh 'docker compose up -d'
               
            }
        }

        stage('Notify') {
            steps {
                // Stuur meldingen (bijv. e-mail) over het resultaat van de pipeline
                sh 'echo "Notifying..."'
            }
        }
    }

    post {
        failure {
            // Voer acties uit bij falen van de pipeline, bijvoorbeeld meldingen of blokkeren van verdere stappen
            sh 'echo "Pipeline failed!"'
        }
    }
}
