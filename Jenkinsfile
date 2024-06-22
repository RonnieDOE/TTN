def registry = 'https://ronnieaishman.jfrog.io'
pipeline {
    agent {
        node {
            label 'maven-slave'
            customWorkspace '/home/ubuntu/jenkins/workspace/Trend_Multi_Branch_Pipeline_main'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.8/bin:${env.PATH}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false, extensions: [],
                    submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/RonnieDOE/TTN.git']]
                ])
            }
        }
        stage('Build') {
            steps {
                echo "############# build started #############"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "############# build completed #############"
            }
        }
        
        stage('Test') {
            steps {
                echo "############# unit test started #############"
                sh 'mvn surefire-report:report'
                echo "############# unit test completed #############"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner -X"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Jar Publish') {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifact-cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'  
                }
            }   
        } 
    }
}