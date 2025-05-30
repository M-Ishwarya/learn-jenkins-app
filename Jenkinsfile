pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'c2177da9-18f4-4754-8d11-cebb1bdce769'
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
                    echo "Building app......"
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

                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }   
            }
            }
        }

        // Deploy in netlify

        stage('Deploy staging') {
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

                   echo "Deploying to staging. SITE ID:$NETLIFY_SITE_ID "

                   node_modules/.bin/netlify status

                   node_modules/.bin/netlify deploy --dir=build 
                '''
            }
        }

        // Adding approval to deploy
        stage('Approval')
        {
            steps{   
                    timeout(time: 15, unit: 'MINUTES')
                    {
                       input message: 'Do you wish to deploy in prod?', ok: 'yes, I am sure!'
                    }
            }
        }


         stage('Deploy prod') {
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

                   node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') 
            {
                agent{
                    docker{
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                        args ''
                    }
                }

                environment
                {
                    CI_ENVIRONMENT_URL = 'https://benevolent-halva-bad07e.netlify.app'
                }

                steps
                {
                        echo "End to End Prod phase.."
                        sh '''
                            npx playwright test --reporter=html
                        '''
                }
                post{
                    always {

                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }   
            }


    }
        
}
