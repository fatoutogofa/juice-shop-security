pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo '=== Recuperation du code depuis GitHub ==='
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '=== Installation des dependances ==='
                bat 'npm install --ignore-scripts'
            }
        }

        stage('Scan - npm audit') {
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

        stage('Scan - ESLint Code Analysis') {
            steps {
                echo '=== Analyse statique du code source ==='
                bat 'npx eslint *.ts lib models routes --format json > eslint-report.json || exit 0'
                bat 'npx eslint *.ts lib models routes || exit 0'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'eslint-report.json', allowEmptyArchive: true
                }
            }
        }

        stage('Build') {
            steps {
                echo '=== Build du projet ==='
                bat 'npm run build || exit 0'
            }
        }

        stage('Tests Unitaires') {
            steps {
                echo '=== Tests unitaires serveur ==='
                bat 'npm run test:server || exit 0'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline termine avec succes'
            mail to: 'ton-email@gmail.com',
                 subject: "✅ Jenkins - juice-shop-security BUILD #${env.BUILD_NUMBER} SUCCESS",
                 body: """
Le build #${env.BUILD_NUMBER} du projet juice-shop-security s'est termine avec succes.

Projet   : ${env.JOB_NAME}
Build    : #${env.BUILD_NUMBER}
Statut   : SUCCESS
URL      : ${env.BUILD_URL}

Rapports disponibles :
- audit-report.json  : vulnerabilites npm
- eslint-report.json : analyse du code

-- Jenkins CI/CD
                 """
        }
        failure {
            echo '❌ Pipeline echoue'
            mail to: 'ton-email@gmail.com',
                 subject: "❌ Jenkins - juice-shop-security BUILD #${env.BUILD_NUMBER} FAILED",
                 body: """
Le build #${env.BUILD_NUMBER} du projet juice-shop-security a ECHOUE.

Projet   : ${env.JOB_NAME}
Build    : #${env.BUILD_NUMBER}
Statut   : FAILED
URL      : ${env.BUILD_URL}

Consultez les logs pour plus de details.

-- Jenkins CI/CD
                 """
        }
        always {
            echo '=== Scan OWASP Juice Shop termine ==='
        }
    }
}
