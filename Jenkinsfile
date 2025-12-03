pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('GitInspector Analysis') {
            steps {
                script {
                    sh '''
                        # Crear directorio para reportes
                        mkdir -p gitinspector-reports
                        
                        # Ejecutar GitInspector con formato HTML
                        gitinspector -f java,js,py,groovy,yaml,yml,html,css,json --format html . > gitinspector-reports/report.html || true
                        
                        echo "GitInspector analysis completed"
                    '''
                }
            }
        }
        
        stage('Publish GitInspector Report') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'gitinspector-reports',
                    reportFiles: 'report.html',
                    reportName: 'GitInspector',
                    reportTitles: 'GitInspector Analysis'
                ])
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
        }
        success {
            echo 'GitInspector analysis successful'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
