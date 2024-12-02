pipeline {
    agent any

    stages {
        /*stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
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
        }*/
        stage('Test'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                } }
            steps{
                echo 'Test Stage'
                sh'npm test'
            }
             post {
        always{
            junit 'test-results/junit.xml'
        }
    }
        }
    
    stage('E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.49.0-noble'
                    reuseNode true
                } }
            steps{
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''
            }
        }
    }
   
}

