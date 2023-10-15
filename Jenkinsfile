pipeline {
    agent any
    environment {
        GITHUB_TOKEN=credentials('github-token')
        IMAGE_NAME='erniedm/etherpad-lite'
        IMAGE_VERSION='latest'
    }

        stages {
            stage('Checkout') {
                steps {
                    // Haal de broncode op uit de Git-repository
                    checkout scm
                }
            }

        stage('Image Cleanup') {
            steps {
                sh 'docker system prune -a --volumes --force'
            }
        }

        stage('ESLint Check') {
            steps {
                script {
                    def eslintError = null // Initialize eslintError

                    try {
                        sh 'rm -rf node_modules package-lock.json && npm install'
                        sh 'rm eslint.xml || true'
                        sh './node_modules/eslint/bin/eslint.js -f checkstyle src > eslint.xml'
                    } catch (Exception e) {
                        // Catch any exception and handle it gracefully
                        eslintError = "ESLint Check failed: ${e.message}"
                        echo eslintError
                        currentBuild.result = 'SUCCESS' // Set overall build result to SUCCESS
                    }

                    archiveArtifacts artifacts: 'eslint.xml', allowEmptyArchive: true
                }
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

        stage ('Trufflehog Check') {
            steps {
                sh 'docker pull trufflesecurity/trufflehog:latest'
                sh 'rm trufflehog_results.json || true'
                sh 'rm trufflehog_results.html || true'
                sh 'docker run -v "/home/s127280/Opdracht1/etherpad-lite:/pwd" trufflesecurity/trufflehog:latest filesystem /pwd --json 2>&1 | grep -v "unable to read file for MIME type detection: EOF" > trufflehog_results.json'
                sh './convert_json_to_html.sh'
                archiveArtifacts artifacts: 'trufflehog_results.html', allowEmptyArchive: true
            }
        }
    
        stage('Build') {
            steps {
                // Voer de buildstappen uit (bijv. installatie van afhankelijkheden, bouwen van Etherpad)
                sh 'docker compose build'
            }
        }

        // stage('Login to GHCR') {
        //     steps {
        //         sh 'echo $GITHUB_TOKEN_PSW | docker login ghcr.io -u $GITHUB_TOKEN_USR --password-stdin'
        //     }
        // }

        // stage ('Tag Image') {
        //     steps {
        //         sh 'docker tag $IMAGE_NAME:$IMAGE_VERSION ghcr.io/$IMAGE_NAME:$IMAGE_VERSION'
        //     }
        // }

        // stage ('Push Image') {
        //     steps {
        //         sh 'docker push ghcr.io/$IMAGE_NAME:$IMAGE_VERSION'
        //     }
        // }

        // stage ('Notation Login') {
        //     steps {
        //         sh 'echo $GITHUB_TOKEN_PSW | notation login ghcr.io -u $GITHUB_TOKEN_USR --password-stdin'
        //     }
        // }

        // stage ('Image Signing') {
        //     steps {
        //         sh 'if [ ! -f /var/lib/jenkins/.config/notation/localkeys/etherpad.org.key ]; then notation cert generate-test --default "etherpad.org"; fi'
        //         // sh 'notation sign ghcr.io/$IMAGE_NAME:$IMAGE_VERSION'
        //         script {
        //             def signedImage = sh(script: 'notation sign ghcr.io/$IMAGE_NAME:$IMAGE_VERSION', returnStdout: true).trim()
        //             echo signedImage
        //             // Extract the image tag from the signedImage string
        //             def indexOfAt = signedImage.lastIndexOf('@')
        //             if (indexOfAt >= 0) {
        //                 // Save the signed image tag in an environment variable.
        //                 env.SIGNED_IMAGE = 'ghcr.io/' + IMAGE_NAME + ':' + IMAGE_VERSION + signedImage.substring(indexOfAt)
        //             } else {
        //                 currentBuild.result = 'FAILURE'
        //                 error("Invalid signed image format: $signedImage")
        //             }                                    
        //         }        
        //     }
        // }

        // stage ('Image Verification') {
        //     steps {
        //         sh 'notation policy import ./trustpolicy.json'
        //         sh 'notation verify $SIGNED_IMAGE'
        //     }
        // }
        stage('Trivy Check') {
            steps {
                sh "trivy image --no-progress --exit-code 0 --severity MEDIUM,HIGH,CRITICAL --format template --template '@/usr/local/share/trivy/templates/html.tpl' -o trivy_report.html secdevops-etherpad:latest"
                archiveArtifacts artifacts: 'trivy_report.html', allowEmptyArchive: true
            }
        }
    }

    post {
        failure {
            sh 'echo "Pipeline failed!"'
        }
        always {
            sh 'docker logout'
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
            recordIssues enabledForFailure: true, aggregatingResults: true, tool: checkStyle(pattern: 'eslint.xml')
        }
    }     
}
