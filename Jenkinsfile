pipeline {
    agent any

    tools {
        nodejs 'NodeJS-22'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '=== Recuperation du code depuis GitHub ==='
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '=== Installation des dependances Node.js ==='
                bat 'npm install --ignore-scripts'
            }
        }

        stage('Scan 1 - npm audit') {
            steps {
                echo '=== Scan des vulnerabilites des dependances ==='
                bat 'npm audit --json > audit-report.json || exit 0'
                bat 'npm audit || exit 0'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'audit-report.json', allowEmptyArchive: true
                }
            }
        }

        stage('Scan 2 - Semgrep') {
            steps {
                echo '=== Analyse statique du code source avec Semgrep ==='
                bat 'semgrep scan --config=auto --json --output=semgrep-report.json . || exit 0'
                bat 'semgrep scan --config=auto . || exit 0'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'semgrep-report.json', allowEmptyArchive: true
                }
            }
        }

        stage('Scan 3 - njsscan') {
            steps {
                echo '=== Scan de securite Node.js avec njsscan ==='
                bat 'njsscan --json -o njsscan-report.json . || exit 0'
                bat 'njsscan . || exit 0'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'njsscan-report.json', allowEmptyArchive: true
                }
            }
        }

        stage('Scan 4 - Retire.js') {
            steps {
                echo '=== Detection des librairies JavaScript vulnerables ==='
                bat 'npx retire --path . --outputformat json --outputpath retire-report.json || exit 0'
                bat 'npx retire --path . || exit 0'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'retire-report.json', allowEmptyArchive: true
                }
            }
        }

    }

    post {
        success {
            echo '=== Pipeline termine avec succes ==='
            mail to: 'fatoutogofatou@gmail.com',
                 subject: "Jenkins - juice-shop-security BUILD #${env.BUILD_NUMBER} SUCCESS",
                 body: """
Le build #${env.BUILD_NUMBER} du projet juice-shop-security s'est termine avec succes.

Projet   : ${env.JOB_NAME}
Build    : #${env.BUILD_NUMBER}
Statut   : SUCCESS
URL      : ${env.BUILD_URL}

Rapports generes :
- audit-report.json    : vulnerabilites npm audit
- semgrep-report.json  : analyse statique Semgrep
- njsscan-report.json  : scan securite Node.js
- retire-report.json   : librairies JS vulnerables

-- Jenkins CI/CD
                 """
        }
        failure {
            echo '=== Pipeline echoue ==='
            mail to: 'fatoutogofatou@gmail.com',
                 subject: "Jenkins - juice-shop-security BUILD #${env.BUILD_NUMBER} FAILED",
                 body: """
Le build #${env.BUILD_NUMBER} du projet juice-shop-security a ECHOUE.

Projet   : ${env.JOB_NAME}
Build    : #${env.BUILD_NUMBER}
Statut   : FAILED
URL      : ${env.BUILD_URL}

Consultez les logs pour plus de details : ${env.BUILD_URL}console

-- Jenkins CI/CD
                 """
        }
        always {
            echo '=== Scan OWASP Juice Shop termine ==='
        }
    }
}
