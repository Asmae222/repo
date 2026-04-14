pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    parameters {
        booleanParam(name: 'SKIP_QUALITY', defaultValue: false, description: 'Passer la qualite statique')
        string(name: 'GIT_COMMIT_SHA', defaultValue: 'main', description: 'Branche ou SHA du commit')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Environnement cible')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Ignorer les tests')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Validation parallele') {
            when {
                expression { return !params.SKIP_TESTS }
            }
            parallel {
                stage('Tests unitaires') {
                    steps {
                        bat 'mvn test -B'
                    }
                    post {
                        always {
                            junit '**/target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Analyse qualite') {
                    steps {
                        bat 'mvn checkstyle:checkstyle pmd:pmd -B'
                    }
                }
                stage('Couverture') {
                    steps {
                        bat 'mvn test jacoco:report -B'
                    }
                }
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
            when {
                expression { return params.ENVIRONMENT == 'prod' }
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Deployer en PRODUCTION ?', ok: 'Oui, continuer'
                }
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
                body: "Le build a reussi !\n\nProjet : ${env.JOB_NAME}\nBuild : #${env.BUILD_NUMBER}\nURL : ${env.BUILD_URL}",
                to: "asmaeelfehri@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "Build FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le build a echoue.\n\nProjet : ${env.JOB_NAME}\nBuild : #${env.BUILD_NUMBER}\nURL : ${env.BUILD_URL}",
                to: "asmaeelfehri@gmail.com"
            )
        }
    }
}