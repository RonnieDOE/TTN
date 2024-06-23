def registry = 'https://ronnieaishman.jfrog.io'
def imageName = 'ronnieaishman.jfrog.io/ronnie-docker-local/ttrend'
def version   = '2.1.3'

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
                    def server = Artifactory.newServer(url: registry + "/artifactory", credentialsId: "artifact_cred")
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "artifactory-libs-release-local/{1}",
                              "flat": "false",
                              "props": "${properties}",
                              "exclusions": ["*.sha1", "*.md5"]
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

        stage('Docker Build') {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName + ":" + version)
                    echo '<--------------- Docker Build Ended --------------->'
                }
            }
        }

        stage('Docker Publish') {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'
                    docker.withRegistry(registry, 'artifact_cred') {
                        app.push()
                    }
                    echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo '<--------------- Helm Deploy Started --------------->'
                    sh 'helm upgrade --install ttrend ttrend-0.1.0.tgz'
                    echo '<--------------- Helm Deploy Ended --------------->'
                }
            }
        }
    }
}