pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    parameters {
        booleanParam(
            name: 'SKIP_QUALITY',
            defaultValue: false,
            description: 'Passer la qualite statique (pour debug rapide)'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build + Tests + Coverage') {
            steps {
                bat 'mvn clean verify -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    junit '**/target/failsafe-reports/*.xml'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Qualite statique') {
            when {
                expression { return !params.SKIP_QUALITY }
            }
            steps {
                bat 'mvn checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs -B'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/checkstyle-result.xml, target/pmd.xml, target/cpd.xml, target/spotbugsXml.xml', allowEmptyArchive: true
                }
            }
        }

        stage('Validation manuelle') {
            steps {
                input message: 'Rapports qualite OK ? Continuer vers le deploiement ?', ok: 'Oui, continuer'
            }
        }

        stage('Trigger job chaine') {
            steps {
                script {
                    build job: 'tp-boutique-postbuild', wait: true, propagate: true
                }
            }
        }

    }

    post {
        success {
            emailext(
                subject: "Build SUCCES - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Le build a reussi !

Projet : ${env.JOB_NAME}
Build : #${env.BUILD_NUMBER}
URL : ${env.BUILD_URL}
""",
                to: "asmaeelfehri@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "Build FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Le build a echoue.

Projet : ${env.JOB_NAME}
Build : #${env.BUILD_NUMBER}
URL : ${env.BUILD_URL}
""",
                to: "asmaeelfehri@gmail.com"
            ) 
        }
    }
}