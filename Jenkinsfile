pipeline {
    agent any
    tools { 
        maven 'Jenkins Maven' 
    }
    stages {
        stage('CI') {
            steps {
                snDevOpsStep()
                sh '''
                    export M2_HOME=/opt/maven/apache-maven-3.6.3 # your Mavan home path
                    export PATH=$PATH:$M2_HOME/bin
                    mvn --version
                '''
                sh 'mvn compile'
                sh 'mvn verify'
            }
            post {
                success {
                    junit '**/target/surefire-reports/*.xml' 
                }
            }
        }
        stage('UAT deploy') {
            steps {
                snDevOpsStep()
                sh '''
                    export M2_HOME=/opt/maven/apache-maven-3.6.3 # your Mavan home path
                    export PATH=$PATH:$M2_HOME/bin
                    mvn --version
                '''
                sh 'mvn package'
                snDevOpsArtifact(artifactsPayload: """{"artifacts": [{"name": "globex-jk.war", "version": "1.0.$BUILD_NUMBER","semanticVersion": "1.0.$BUILD_NUMBER","repositoryName": "CorpSite-jenkins"}]}""")

                sh 'mv /opt/tomcat/webapps/globex-web.war /opt/tomcat/webapps/globex-web.war.tmp'
                
                script {
                    sshPublisher(continueOnError: false, failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName:'CorpSite UAT',
                            verbose: true,
                            transfers: [
                                
                                sshTransfer(
                                    sourceFiles: 'target/globex-web.war',
                                    removePrefix: 'target/',
                                    remoteDirectory: 'tomcat/webapps'
                                )
                            ]
                        )
                    ])
                }
                
                sh 'mv /opt/tomcat/webapps/globex-web.war /opt/tomcat/webapps/globex-web-uat.war'
                sh 'mv /opt/tomcat/webapps/globex-web.war.tmp /opt/tomcat/webapps/globex-web.war'
            }
        }
        stage('UAT test') {
            steps {
                snDevOpsStep()
                sh '''
                    export M2_HOME=/opt/maven/apache-maven-3.6.3 # your Mavan home path
                    export PATH=$PATH:$M2_HOME/bin
                    mvn --version
                '''
                sh 'mvn compile'
                sh 'mvn verify'
            }
            post {
                success {
                    junit '**/target/surefire-reports/*.xml' 
                }
            }
            /*
            steps {
                sh '''
                    export M2_HOME=/opt/maven/apache-maven-3.6.3 # your Mavan home path
                    export PATH=$PATH:$M2_HOME/bin
                    mvn --version
                '''
                sh 'mvn compile'
          
                sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=CorpSite \
                    -Dsonar.host.url=http://sonarqube.sndevops.xyz:9000 \
                    -Dsonar.login=efef5144be738a606c23fff3f139f00965b82869 \
                    -Dsonar.exclusions=src/main/webapp/resources/js/bootstrap.js \
                    -Dsonar.analysis.scm=$GIT_COMMIT \
                    -Dsonar.analysis.buildURL=$BUILD_URL
                '''
               
            }
            */
        }
        stage('deploy') {
            steps {
                snDevOpsStep()
                snDevOpsPackage(name: "Globex-package", artifactsPayload: """{"artifacts": [{"name": "globex-jk.war", "version": "1.0.$BUILD_NUMBER","repositoryName": "CorpSite-jenkins"}]}""")
                snDevOpsChange()
                script {
                    sshPublisher(continueOnError: false, failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName:'CorpSite PROD',
                            verbose: true,
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'target/globex-web.war',
                                    removePrefix: 'target/',
                                    remoteDirectory: 'tomcat/webapps'
                                )
                            ]
                        )
                    ])
                }
            }
        }
    }
}
