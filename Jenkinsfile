pipeline {
    agent {
        node {
            label 'maven-slave'
        }
    }
 
    environment {
        PATH = "/opt/apache-maven-3.9.8/bin:${env.PATH}"
    }

    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }

    stage('SonarQube analysis') {
    environment {
      scannerHome = tool 'sonar-scanner'
    }
    steps{
    withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
      sh "${scannerHome}/bin/sonar-scanner"
    }
    }
  }
}
}