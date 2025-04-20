pipeline {
    agent any
    stages {
        stage('checking out from git') {
            steps {
                echo "connecting to git"
                git branch: 'prod', url: 'https://github.com/SharmaPalani/enahanced-petclinc-springboot.git'
            }
        }
        stage('maven compile') {
            steps {
                echo "maven compile"
                sh 'mvn compile'
            }
        }
    }
}