pipeline {
    agent any
    tools {
        maven 'Maven-3.6.3'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Now Archiving war...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage ('Docker') {
            steps {
                sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                sh "docker save -o webapp.docker tomcatwebapp:${env.BUILD_ID}"
            }
            post {
                success {
                    echo 'Now Archiving docker image...'
                    archiveArtifacts artifacts: 'webapp.docker'
                    sh 'rm webapp.docker'
                }
            }
        }
        stage ('Deploy to Staging') {
            steps {
                parallel(
                    deploy: {
                        build job: 'deploy-to-staging-maven-project'
                    },
                    check: {
                        build job: 'static-analysis-maven-project'
                    }
                )
            }
        }
        stage ('Deploy to Production') {
            steps {
                timeout(time:5, unit:'DAYS') {
                    input message: 'Approve PRODUCTION Deployment?'
                }
                build job: 'deploy-to-prod-maven-project'
            }
            post {
                success {
                    echo 'Code deployed to Production.'
                }

                failure {
                    echo ' Deployment failed.'
                }
            }
        }
    }
}
