pipeline {
    
	agent any

	tools {
        maven "maven"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "52.91.8.109:8081"
        NEXUS_REPOSITORY = "ce-release"
	    NEXUS_REPOGRP_ID    = "ce-group"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
    }
	
    stages{
        
        stage('BUILD') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                    slackSend channel: '#jenkins', color: 'good', message: "Build stage succeeded!"
                }
                failure {
                    echo 'Build stage failed!'
                    slackSend channel: '#jenkins', color: 'danger', message: "Build stage failed!"
                }
            }
        }
    }

    stage('UNIT TEST') {
        steps {
            sh 'mvn test'
        }
        post {
            success {
                echo 'Unit Test stage succeeded!'
                slackSend channel: '#jenkins', color: 'good', message: "Unit Test stage succeeded!"
            }
            failure {
                echo 'Unit Test stage failed!'
                slackSend channel: '#jenkins', color: 'danger', message: "Unit Test stage failed!"
            }
        }
    }

    stage('INTEGRATION TEST') {
        steps {
            sh 'mvn verify -DskipUnitTests'
        }
        post {
            success {
                echo 'Integration Test stage succeeded!'
                slackSend channel: '#jenkins', color: 'good', message: "Integration Test stage succeeded!"
            }
            failure {
                echo 'Integration Test stage failed!'
                slackSend channel: '#jenkins', color: 'danger', message: "Integration Test stage failed!"
            }
        }
    }

    stage('CODE ANALYSIS WITH CHECKSTYLE') {
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Code Analysis with Checkstyle stage succeeded!'
                slackSend channel: '#jenkins', color: 'good', message: "Code Analysis with Checkstyle stage succeeded!"
            }
            failure {
                echo 'Code Analysis with Checkstyle stage failed!'
                slackSend channel: '#jenkins', color: 'danger', message: "Code Analysis with Checkstyle stage failed!"
            }
        }
    }

    stage('CODE ANALYSIS with SONARQUBE') {
        environment {
            scannerHome = tool 'sonarscanner'
        }
        steps {
            withSonarQubeEnv('sonar-pro') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ce \
                    -Dsonar.projectName=ce \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
        post {
            success {
                echo 'Code Analysis with SonarQube stage succeeded!'
                slackSend channel: '#jenkins', color: 'good', message: "Code Analysis with SonarQube stage succeeded!"
            }
            failure {
                echo 'Code Analysis with SonarQube stage failed!'
                slackSend channel: '#jenkins', color: 'danger', message: "Code Analysis with SonarQube stage failed!"
            }
        }
    }

    stage("Publish to Nexus Repository Manager") {
        steps {
            script {
                pom = readMavenPom file: "pom.xml";
                filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                artifactPath = filesByGlob[0].path;
                artifactExists = fileExists artifactPath;
                if (artifactExists) {
                    echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: NEXUS_REPOGRP_ID,
                        version: ARTVERSION,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging],
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: "pom.xml",
                            type: "pom"]
                        ]
                    );
                } else {
                    error "*** File: ${artifactPath}, could not be found";
                }
            }
        }
        post {
            success {
                echo 'Publish to Nexus Repository Manager stage succeeded!'
                slackSend channel: '#jenkins', color: 'good', message: "Publish to Nexus Repository Manager stage succeeded!"
            }
            failure {
                echo 'Publish to Nexus Repository Manager stage failed!'
                slackSend channel: '#jenkins', color: 'danger', message: "Publish to Nexus Repository Manager stage failed!"
            }
        }
    }
    }