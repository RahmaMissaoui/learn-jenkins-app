pipeline {
    agent any

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
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
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
                            args '-u root'
                        }
                    }
                    steps {
                        sh '''
                            # test -f build/index.html
                            npm test
                        '''
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-u root'          
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            # Start server and get PID
                            node_modules/.bin/serve -s build -l 4000 &
                            SERVER_PID=$!
                            # Wait for server to be ready
                            npx wait-on http://localhost:4000 || sleep 15
                            # Run tests
                            npx playwright test --reporter=html
                            # Kill the server
                            kill $SERVER_PID
                        '''
                    }
                    post {
                        always {
                            archiveArtifacts 'playwright-report/**/*'
                        }
                    }
                }
            }  // This closes the parallel block
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
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }  
}