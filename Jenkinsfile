pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
    }

    environment {
        BACKEND_DIR = 'crud_backend/crud_backend-main'
        FRONTEND_DIR = 'crud_frontend/crud_frontend-main'

        TOMCAT_URL = 'http://16.16.206.169:9090/manager/text'
        TOMCAT_CREDS = 'tomcat-creds'   // Jenkins credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/geethakandepi/fullstack.git', branch: 'main'
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh '''
                        echo "=== Install frontend deps ==="
                        npm ci
                        echo "=== Build frontend ==="
                        npm run build
                        echo "=== Verify dist ==="
                        ls -la dist || { echo "No dist folder found"; exit 1; }
                    '''
                }
            }
        }

        stage('Integrate Frontend into Backend') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh '''
                        echo "=== Clean old static ==="
                        rm -rf src/main/resources/static/* || true
                        echo "=== Copy frontend build into backend static ==="
                        cp -r ../../${FRONTEND_DIR}/dist/. src/main/resources/static/
                        ls -la src/main/resources/static || true
                    '''
                }
            }
        }

        stage('Build Backend WAR') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh '''
                        echo "=== Package backend ==="
                        mvn -B clean package -DskipTests
                        echo "=== Verify WAR ==="
                        ls -la target/*.war
                    '''
                }
            }
        }

        stage('Deploy Backend WAR to Tomcat') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    script {
                        def warFile = sh(script: "ls -1 target/*.war | head -n1", returnStdout: true).trim()
                        if (!warFile) {
                            error "No WAR found in target/"
                        }
                        withCredentials([usernamePassword(credentialsId: env.TOMCAT_CREDS, usernameVariable: 'TUSER', passwordVariable: 'TPASS')]) {
                            sh """
                                echo "=== Deploying WAR to Tomcat ==="
                                curl -v -u "$TUSER:$TPASS" --upload-file "$warFile" \
                                  "${TOMCAT_URL}/deploy?path=/springapp1&update=true"
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Application deployed: http://16.16.206.169:9090/springapp1"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
