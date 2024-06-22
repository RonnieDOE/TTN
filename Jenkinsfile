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
        stage("test"){
            steps{
                echo "############# unit test started #############"
                sh 'mvn surefire-report:report'
                echo "############# unit test completed #############"
            }
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
    }
}