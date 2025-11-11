pipeline {
    agent none

    environment {
        NETLIFY_SITE_ID = 'your-netlify-site-id-here'  // Replace with your actual Site ID or use Jenkins env var
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
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
                            image 'node:18-bookworm-slim'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            apt-get update && apt-get install -y ca-certificates curl gnupg
                            npm install @playwright/test
                            npx playwright install --with-deps
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
                NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')  // Your secret credential ID
            }
            steps {
                sh '''
                    npm install -g netlify-cli@20.1.1
                    netlify --version
                    echo "Deploying to production..."
                    netlify deploy --dir=build --prod --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --message "Jenkins auto-deploy $(date)"
                '''
            }
        }
    }
}