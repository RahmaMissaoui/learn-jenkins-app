pipeline {
    agent none

    environment {
        NETLIFY_SITE_ID = 'your-site-id-here'  // ← change this in Jenkins Global Environment Variables or credentials
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la build/
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
                            image 'mcr.microsoft.com/playwright:focal'   // ← contains Node.js + Playwright
                            reuseNode true
                            args '-u 0:0 --cap-add=SYS_ADMIN'
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
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')  // ← add this secret in Jenkins Credentials
            }
            steps {
                sh '''
                    npm install -g netlify-cli@20.1.1
                    netlify --version
                    netlify deploy --dir=build --prod --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --message "Jenkins deploy $(date)"
                '''
            }
        }
    }
}