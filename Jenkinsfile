pipeline {
    agent any
    
    environment{
        NETLIFY_SITE_ID = 'e0101d28-7db6-4a72-bb45-e025addaf313'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

   stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                    '''
            }
        }
        stage('Tests'){
            parallel{
                stage('Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        } }
                    steps{
                        
                        echo 'Test Stage'
                        sh''' 
                        npm test 
                         '''
                    }
                    post {
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
            
            stage('E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        } 
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                post {
                    always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                    
                        }
                    }
                }
            }
        }
       
       stage(' Deploy Staging') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    
                    }
                    environment{
                        CI_ENVIRONMENT_URL= 'NULL'
                       
                    }
                    steps{
                        sh '''npm install --save-dev netlify-cli node-jq
                        node_modules/.bin/netlify --version
                        echo "Deploying to production . Site ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                        CI_ENVIRONMENT_URL = $(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                        npx playwright test --reporter=html
                            
                        '''
                    }
                post {
                    always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging', reportTitles: '', useWrapperFileDirectly: true])
                    
                        }
     
   
                    }
               }
       
       stage('Approval'){
           steps{
              timeout(15) {
                 input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
               
           }
       }
       
       stage('Deploy Prod') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    
                    }
                    environment{
                       CI_ENVIRONMENT_URL = 'https://venerable-froyo-f2d7c4.netlify.app' 
                    }
                    steps{
                        sh '''
                            npm install --save-dev netlify-cli 
                            node_modules/.bin/netlify --version
                            echo "Deploying to production . Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod
                            npx playwright test --reporter=html
                            
                        '''
                    }
                post {
                    always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                    
                        }
     
   
                    }
               }
         }
    }
