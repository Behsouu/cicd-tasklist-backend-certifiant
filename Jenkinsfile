pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'SonarScanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment Check') {
            // Diagnostic : révèle immédiatement si Node.js manque sur l'agent,
            // plutôt que de le découvrir au milieu d'un stage npm plus tard.
            steps {
                sh '''
                    echo "--- Node ---"
                    node --version || echo "NODE NON TROUVE"
                    echo "--- npm ---"
                    npm --version || echo "NPM NON TROUVE"
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm run test:coverage'
            }
            post {
                always {
                    // Publie les résultats même si les tests échouent,
                    // pour voir le détail dans l'UI Jenkins (exigence C17.1)
                    junit 'reports/junit.xml'
                }
            }
        }

        stage('E2E Tests') {
            steps {
                sh 'npm run test:e2e:coverage'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server-1') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Timeout de sécurité : si le webhook Sonar->Jenkins ne répond
                // jamais (le blocage qu'on a eu en TP5), le pipeline échoue
                // proprement après 5 min au lieu de rester bloqué indéfiniment.
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}