pipeline {
    agent none

    environment {
        NETLIFY_SITE_ID = 'your-netlify-site-id-here'  // ← replace or add as Jenkins env var
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'   // ← this solves the EACCES permission error forever
                }
            }
            steps {
                sh '''
                    npm ci --legacy-peer-deps
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps {
                        sh 'npm test'
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:focal'
                            reuseNode true
                            args '-u root:root --cap-add=SYS_ADMIN'
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright E2E Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')  // ← Secret text credential ID
            }
            steps {
                sh '''
                    npm install -g netlify-cli@20.1.1
                    netlify --version
                    echo "Deploying to production..."
                    netlify deploy --dir=build --prod --message "Jenkins auto-deploy $(date)"
                '''
            }
        }
    }
}