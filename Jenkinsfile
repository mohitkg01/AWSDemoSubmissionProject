pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    sh './gradlew clean build --stacktrace -i'
                }
            }
        }
        
        stage('Publish') {
            steps {
                script {
                    sh """
                        ./gradlew -i --stacktrace publish \
                            -PMVN_USERNAME=${MVN_USERNAME} \
                            -PMVN_PASSWORD=${MVN_PASSWORD} \
                            -PMVN_VERSION=1.${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Post') {
            steps {
                jacoco()
                junit 'lib/build/test-results/test/*.xml'
                def pmd = scanForIssues tool: [$class: 'Pmd'], pattern: 'lib/build/reports/pmd/*.xml'
                publishIssues issues: [pmd]
            }
        }

        stage('Publish Build Info') {
            environment {
                JFROG_CLI_OFFER_CONFIG = false
            }
            steps {
                container('jfrog-cli-go') {
                    withCredentials([usernamePassword(credentialsId: 'gociexamplerepo', passwordVariable: 'APIKEY', usernameVariable: 'USER')]) {
                        sh "jfrog rt bce $JOB_NAME $BUILD_NUMBER"
                        sh "jfrog rt bag $JOB_NAME $BUILD_NUMBER"
                        sh "jfrog rt bad $JOB_NAME $BUILD_NUMBER \"go.*\""
                        sh "jfrog rt bp --build-url=https://jenkins.openshiftk8s.com/ --url=https://partnership.jfrog.io/artifactory --user=$USER --apikey=$APIKEY $JOB_NAME $BUILD_NUMBER"
                    }
                }
            }
        }
    }
}
