pipeline {
    agent any

    triggers {
        // Trigger build when GitHub/GitLab/Bitbucket webhook fires
        githubPush()
        // pollSCM('H/5 * * * *') // fallback: poll every 5 minutes if webhook not working
    }

    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('git-checkout') {
            steps {
                script {
                    git branch: 'dev', url: 'https://github.com/Elvis-Ikay/JavaWeb3.git'
                }
            }
        }
        stage('build') {
            steps {
                script {
                    sh 'mvn clean install'
                    sh "cp target/WebAppCal-0.0.1.war target/WebAppCal-${BUILD_NUMBER}.war"
                }
            }
        }
        stage('code-analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                                -Dsonar.projectKey=webapp \
                                -Dsonar.projectName=webapp\
                                -Dsonar.host.url=http://54.242.229.73:9000\
                                -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }
        stage('nexus-uploader') {
            steps {
                script {

                    def warFile = findFiles(glob: 'target/*.war')[0].path

                    nexusArtifactUploader(
                        artifacts: [[
                            artifactId: 'web',
                            classifier: '',
                            file: warFile,
                            type: 'war'
                        ]],
                        credentialsId: 'nexus-creds',
                        groupId: 'webapp',
                        nexusUrl: '54.205.2.82:8081/',
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        repository: 'artifacts',
                        version: "${BUILD_NUMBER}"
                    )
                }
            }
        }
        stage('deploy-to-tomcat') {
            steps {
                script {
                    deploy adapters: [tomcat9(
                        alternativeDeploymentContext: '',
                        credentialsId: 'nexusandtomcat',
                        path: '',
                        url: 'http://52.91.10.193:8080/manager/text'
                    )], war: '**/*.war'
                }
            }
        }
    }
}
