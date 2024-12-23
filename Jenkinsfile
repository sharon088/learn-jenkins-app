pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID= '6c0603b1-b009-4741-8d76-1f7dc0074868'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                    ls -la 
                    npm ci
                    npm run build
                '''
            }
        }
        stage('Run Tests')
            {
                parallel{
                    stage('Test')
                    {
                        agent{
                            docker{
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                        steps{
                            sh '''
                                test -f build/index.html
                                npm test
                            '''
                        }
                    }

                    stage('E2E') {
                        steps {
                        sh '''
                            sleep 10
                        '''
                        }
                    }

                }
            }   

        stage('Deploy To Staging')
        {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir==build --json > deploy-output.json
                    
                '''
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",returnStdout: true)
                }
            }
        }

        stage('pass variable')
        {
            environment {
                CI_ENVIRONMENT_URL = "$(env.STAGING_URL)"
            }
            steps {
                sh '''
                    cho "CI_ENVIRONMENT_URL: $CI_ENVIRONMENT_URL"
                '''
            }
        }
    
        stage('Approval') {
            steps {
                timeout(time:15,unit: 'MINUTES')
                {
                    input message: 'Do you want deploy to production ? ', ok: 'Yes i am sure !'
                }
            }
        }

        stage('Deploy')
        {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli 
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir==build --prod
                '''
            }
        }

        }

    post {
        always {
            echo "hello world"
        }
    }
}
