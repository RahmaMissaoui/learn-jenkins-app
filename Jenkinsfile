pipeline {
    agent none

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root'
                }
            }
            steps {
                sh 'npm ci --legacy-peer-deps'
                sh 'npm run build'
                sh 'chmod -R 777 test-results/'   // يحل مشكلة الكتابة في junit.xml
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                            args '-u root'
                        }
                    }
                    steps {
                        sh 'npm test -- --ci --reporters=jest-junit --reporters=default'
                    }
                    post {
                        always {
                            junit keepLongStdio: true, testResults: 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:focal'
                            reuseNode true
                            args '-u root --cap-add=SYS_ADMIN'
                        }
                    }
                    steps {
                        sh '''
                            npm ci --legacy-peer-deps
                            npm install serve
                            chmod -R 777 node_modules/
                            node_modules/.bin/serve -s build &
                            sleep 15
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report'])
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
                    args '-u root'
                }
            }
            steps {
                sh '''
                    npm install -g netlify-cli@20.1.1
                    netlify --version
                '''
            }
        }
    }
}