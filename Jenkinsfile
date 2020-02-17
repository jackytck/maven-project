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
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
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
    }
}
