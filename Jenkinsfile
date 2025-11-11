pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh 'npm install'
                sh 'npm run build'  // This was missing!
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh 'npx playwright install'  // Install browsers
                sh '''
                    npm install serve
                    # Start server in background and get PID
                    node_modules/.bin/serve -s build -p 3000 &
                    SERVER_PID=$!
                    # Wait for server to be ready
                    npx wait-on http://localhost:3000
                    # Run tests
                    npx playwright test --reporter=html
                    # Kill the server
                    kill $SERVER_PID
                '''
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
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

    post {
        always {
            // Make sure to archive the test results
            junit 'test-results/**/*.xml'  // Updated path pattern
            archiveArtifacts 'playwright-report/**/*'  // Archive HTML reports
        }
    }
}