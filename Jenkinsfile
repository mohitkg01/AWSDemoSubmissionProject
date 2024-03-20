pipeline {
    agent any

    environment {
        MVN_USERNAME = credentials('your_mvn_username_id')
        MVN_PASSWORD = credentials('your_mvn_password_id')
    }

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
    }
}
