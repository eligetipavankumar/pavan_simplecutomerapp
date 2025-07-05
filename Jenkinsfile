pipeline {
    agent any

    stages {

        stage('Git Clone') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/eligetipavankumar/sabear_simplecutomerapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'admin-nexus', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                    configFileProvider([configFile(fileId: 'nexus-maven-settings', targetLocation: 'settings.xml')]) {
                        sh '''
                            mvn deploy \
                            -DaltDeploymentRepository=nexus::default::http://13.232.26.18:8081/repository/simple_customer_app/ \
                            --settings settings.xml
                        '''
                    }
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#ci-cd',
                    message: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    tokenCredentialId: 'slack-token'
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-server', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        curl -T target/*.war "http://${TOMCAT_USER}:${TOMCAT_PASS}@13.234.204.49:8080/manager/text/deploy?path=/customerapp&update=true"
                    '''
                }
            }
        }
    }

    post {
        failure {
            slackSend(
                channel: '#ci-cd',
                message: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                tokenCredentialId: 'slack-token'
            )
        }
    }
}
