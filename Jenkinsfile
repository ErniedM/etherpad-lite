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
                    script {
                        def additionalArguments = '''\
                            -o ./
                            -s ./
                            -f ALL
                            --prettyPrint
                        '''

                        dependencyCheck(
                            additionalArguments: additionalArguments,
                            odcInstallation: 'owasp'
                        )
                    }

                    dependencyCheckPublisher(pattern: 'dependency-check-report.xml')
                }
            }

        stage('Start Trufflehog Container') {
            steps {
                script {
                    // Start the Docker container and capture the container ID
                    def dockerImage = docker.image('trufflesecurity/trufflehog:latest')
                    def container = dockerImage.run("-v /home/s127280/Opdracht1/etherpad-lite:/pwd", "--json")
                    def containerId = container.id

                    // Store the container ID in an environment variable for later use
                    env.CONTAINER_ID = containerId
                }
            }
        }

        stage('Trufflehog Check') {
            steps {
                script {
                    // Use the stored container ID to run the scan inside the container
                    sh 'rm trufflehog_results.json || true'
                    sh 'rm trufflehog_results.html || true'
                    sh "docker exec ${env.CONTAINER_ID} trufflehog filesystem /pwd --json 2>&1 | grep -v 'unable to read file for MIME type detection: EOF' > trufflehog_results.json"
                    sh './convert_json_to_html.sh'
                    archiveArtifacts artifacts: 'trufflehog_results.html', allowEmptyArchive: true
                }
            }
        }

        //stage ('Trufflehog Check') {
        //    steps {
        //        sh 'rm trufflehog_results.json || true'
        //        sh 'rm trufflehog_results.html || true'
        //        sh 'trufflehog filesystem /pwd --json 2>&1 | grep -v "unable to read file for MIME type detection: EOF" > trufflehog_results.json'
        //        sh './convert_json_to_html.sh'
        //        archiveArtifacts artifacts: 'trufflehog_results.html', allowEmptyArchive: true
        //    }
        //}
    
        stage('Build') {
            steps {
                // Voer de buildstappen uit (bijv. installatie van afhankelijkheden, bouwen van Etherpad)
                sh 'docker compose build'
            }
        }

        stage('Trivy Scanning') {
            steps {
                sh "trivy image --no-progress --exit-code 0 --severity MEDIUM,HIGH,CRITICAL --format template --template '@/usr/local/share/trivy/templates/html.tpl' -o trivy_report.html secdevops-etherpad:latest"
                archiveArtifacts artifacts: 'trivy_report.html', allowEmptyArchive: true
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
        always {
            publishHTML(
                target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: '.',
                    reportFiles: 'trivy_report.html',
                    reportName: 'Trivy Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ]
            )
            
            publishHTML(
                target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: '.',
                    reportFiles: 'trufflehog_results.html',
                    reportName: 'Trufflehog Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ]
            )
        }
    }     
}
