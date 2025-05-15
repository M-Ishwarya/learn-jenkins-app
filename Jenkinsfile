pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'c2177da9-18f4-4754-8d11-cebb1bdce769'
        NETLIFY_TOKEN = credentials('netlify-token')
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
                    echo "Building app.."
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }


        // Run stages in parallel
        stage('Run Tests'){
            parallel{
                stage('Unit Test')
            {
                agent{
                    docker{
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
                steps
                {
                        echo "Testing phase.."
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                }

                post{
                    always {
                        junit 'jest-results/junit.xml'
                    }
                }   
            }

            stage('E2E') 
            {
                agent{
                    docker{
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                        args ''
                    }
                }
                steps
                {
                        echo "End to End phase.."
                        sh '''

                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                }
                post{
                    always {

                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }   
            }
            }
        }

        // Deploy in netlify
         stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli@20.1.1
                   node_modules/.bin/netlify --version

                   echo "Deploying to production. SITE ID:$NETLIFY_SITE_ID "

                   node_modules/.bin/netlify status
                '''
            }
        }


    }
        
}
