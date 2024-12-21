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
                            resuseNode true
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
                
            }
    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
