pipeline {
    agent any
    stages {
        stage('checking out from git') {
            steps {
                git branch: 'prod', url: 'https://github.com/SharmaPalani/enahanced-petclinc-springboot.git'
            }
        }
    }
}