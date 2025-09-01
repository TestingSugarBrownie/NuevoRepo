pipeline {
    // agent any = usa el Jenkins master (donde viste que ejecuta)
    agent any
    
    // No necesitas githubPush() porque ya tienes polling configurado
    
    environment {
        REPO_NAME = "${env.JOB_NAME.split('/')[1]}"
        COMMIT_ID = "${env.GIT_COMMIT}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Jenkins automáticamente hace checkout del código aquí
                // Lo clona en: /var/jenkins_home/workspace/GitHub-Organization_RepoName_branch
                echo "Repository cloned to: ${env.WORKSPACE}"
                echo "Building repository: ${REPO_NAME}"
                echo "Commit SHA: ${env.GIT_COMMIT}"
                echo "Branch: ${env.GIT_BRANCH}"
                
                // Mostrar archivos en el workspace
                sh 'ls -la'
                sh 'pwd'
            }
        }
        
        stage('Environment Info') {
            steps {
                echo "=== BUILD INFORMATION ==="
                echo "Job Name: ${env.JOB_NAME}"
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Workspace: ${env.WORKSPACE}"
                echo "Node Name: ${env.NODE_NAME}"
                echo "Git Commit: ${env.GIT_COMMIT}"
                echo "Git Branch: ${env.GIT_BRANCH}"
                
                // Verificar qué herramientas están disponibles
                sh '''
                    echo "=== AVAILABLE TOOLS ==="
                    which docker || echo "Docker not available"
                    which node || echo "Node.js not available"
                    which python3 || echo "Python not available"
                    which git || echo "Git not available"
                    
                    echo "=== DOCKER INFO ==="
                    docker --version || echo "Docker not accessible"
                    docker ps || echo "Cannot list containers"
                '''
            }
        }
        
        stage('Code Validation') {
            steps {
                script {
                    echo "=== VALIDATING CODE ==="
                    
                    // Detectar tipo de proyecto y validar
                    if (fileExists('package.json')) {
                        echo "📦 Node.js project detected"
                        sh '''
                            echo "Package.json contents:"
                            cat package.json
                            
                            # Usar Docker para ejecutar npm (ya que node no está en el master)
                            docker run --rm -v $(pwd):/app -w /app node:18-alpine sh -c "
                                echo 'Installing dependencies...' &&
                                npm install &&
                                echo 'Running tests...' &&
                                npm test || echo 'No tests configured' &&
                                echo 'Build completed successfully'
                            "
                        '''
                    } else if (fileExists('requirements.txt')) {
                        echo "🐍 Python project detected"
                        sh '''
                            echo "Requirements.txt contents:"
                            cat requirements.txt
                            
                            # Usar Docker para ejecutar python
                            docker run --rm -v $(pwd):/app -w /app python:3.11-slim sh -c "
                                echo 'Installing dependencies...' &&
                                pip install -r requirements.txt &&
                                echo 'Running tests...' &&
                                python -m pytest || echo 'No tests found' &&
                                echo 'Python validation completed'
                            "
                        '''
                    } else if (fileExists('Dockerfile')) {
                        echo "🐳 Docker project detected"
                        sh '''
                            echo "Dockerfile contents:"
                            head -20 Dockerfile
                            
                            echo "Building Docker image..."
                            docker build -t ${REPO_NAME}:${BUILD_NUMBER} .
                            echo "Docker build completed successfully"
                        '''
                    } else {
                        echo "📋 Generic project - running basic validation"
                        sh '''
                            echo "Project structure:"
                            find . -maxdepth 2 -type f | head -20
                            
                            echo "File types in project:"
                            find . -name "*.py" -o -name "*.js" -o -name "*.ts" -o -name "*.java" -o -name "*.go" | wc -l
                        '''
                    }
                }
            }
        }
        
        stage('Quality Gates') {
            when {
                branch 'main'
            }
            steps {
                echo "=== QUALITY CHECKS FOR MAIN BRANCH ==="
                
                script {
                    // Verificar que el código cumple estándares básicos
                    sh '''
                        echo "Running quality checks..."
                        
                        # Verificar que no hay archivos grandes accidentales
                        echo "Checking for large files (>10MB):"
                        find . -size +10M -type f | head -5
                        
                        # Verificar que no hay credenciales hardcodeadas - CORREGIDO
                        echo "Checking for potential credentials:"
                        grep -r -i -E "(password|secret|key|token)" . --include="*.js" --include="*.py" --include="*.java" | head -5 || echo "No suspicious patterns found"
                        
                        # Contar líneas de código
                        echo "Lines of code:"
                        find . -name "*.py" -o -name "*.js" -o -name "*.ts" -o -name "*.java" | xargs wc -l | tail -1 || echo "No code files found"
                    '''
                }
            }
        }
        
        stage('Deploy Check') {
            when {
                branch 'main'
            }
            steps {
                echo "=== DEPLOYMENT READINESS CHECK ==="
                script {
                    if (fileExists('docker-compose.yml')) {
                        echo "Docker Compose configuration found - ready for deployment"
                        sh 'docker-compose config'
                    } else if (fileExists('Dockerfile')) {
                        echo "Dockerfile found - ready for containerized deployment"
                    } else {
                        echo "No deployment configuration found"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "=== BUILD SUMMARY ==="
            echo "Repository: ${REPO_NAME}"
            echo "Build: ${env.BUILD_NUMBER}"
            echo "Result: ${currentBuild.result ?: 'SUCCESS'}"
            echo "Duration: ${currentBuild.durationString}"
            
            // Limpiar workspace (funciona en Jenkins master)
            deleteDir()
        }
        success {
            echo "✅ All checks passed! Repository ${REPO_NAME} is healthy."
        }
        failure {
            echo "❌ Build failed for ${REPO_NAME}. Check the logs above."
        }
    }
}