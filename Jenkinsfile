pipeline {
    agent any
    
    environment {
        PATH = "${SONAR_SCANNER_HOME}/bin:${env.PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📦 Clonando el repositorio...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Construyendo el proyecto...'
                script {
                    // Adapta este comando según tu tipo de proyecto
                    // Para Java/Maven: sh 'mvn clean package'
                    // Para Python: sh 'pip install -r requirements.txt'
                    // Para Node.js: sh 'npm install && npm run build'
                    sh 'echo "Build completado"'
                }
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Ejecutando tests...'
                script {
                    // Adapta según tu proyecto
                    // Para Java/Maven: sh 'mvn test'
                    // Para Python: sh 'pytest'
                    // Para Node.js: sh 'npm test'
                    sh 'echo "Tests completados"'
                }
            }
        }
        
        
        stage('GitInspector Report') {
            steps {
                echo '📊 Generando reporte de GitInspector...'
                script {
                    sh '''
                        gitinspector -f java,py,js,jsx,ts,tsx,html,css,xml,gradle,kt,swift \
                        --format=html \
                        --grading \
                        --timeline \
                        -w \
                        > gitinspector-report.html || echo "GitInspector completado con advertencias"
                    '''
                }
            }
        }
        
        stage('Publish Reports') {
            steps {
                echo '📄 Publicando reportes...'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'gitinspector-report.html',
                    reportName: 'GitInspector Report',
                    reportTitles: 'Análisis de Código con GitInspector'
                ])
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline ejecutado exitosamente!'
        }
        failure {
            echo '❌ El pipeline falló.'
        }
        always {
            echo '🧹 Limpiando workspace...'
            cleanWs()
        }
    }
}
