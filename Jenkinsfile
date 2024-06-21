pipeline {
    agent {
        node {
            label 'maven-slave'
        }
    }
 
    environment {
        PATH+EXTRA = '/opt/apache-maven-3.9.8/bin'
    }

    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
