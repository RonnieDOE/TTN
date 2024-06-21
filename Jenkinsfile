pipeline {
    agent {
        node {
            label 'maven-slave'
        }
        
    }
enviroment {
    PATH = "/opt/apache-maven-3.9.8/bin:$PATH"
}
    stages {
        stage("build"){
            steps{
                sh 'mvn clean deploy'
            }
        }
    }
}
