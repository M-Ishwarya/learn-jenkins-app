pipeline {
    agent any

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

        stage('Test')
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
        }

        // Playwright
        stage('E2E') // End to End
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
                        #npm install -g serve
                        npm install serve
                        
                        #serve -s build
                        node_modules\.bin\serve -s build

                        npx playwright test
                    '''
            }
        }


    }

    post{
        always {
            junit 'test-results/junit.xml'
        }
    }
        
}
