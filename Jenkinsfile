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
                sh 'mvn clean deploy'
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