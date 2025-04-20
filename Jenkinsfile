pipeline {
    agent any
    tools {
        maven 'maven'
    }
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
        stage('maven test') {
            steps {
                echo "maven test"
                sh 'mvn test'
            }
        }
        stage('filesystem scanning by trivy') {
            steps {
                echo "trivy scanning"
                sh 'trivy fs --format table --output result1.txt .'
            }
        }
        stage('sonar analysis') {
            environment {
                SCANNER_HOME = tool 'Sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                    $SCANNER_HOME/bin/sonar/sonar-scanner \
                    -Dsonar.projectKey=sharmapalani_jenkins-project-pet \
                    -Dsonar.projectName=jenkins_project_pet \
                    -Dsonar.organization=sharmapalani \
                    -Dsonar.java.binaries=. \
                    '''
                }
            }
        }

    }
}