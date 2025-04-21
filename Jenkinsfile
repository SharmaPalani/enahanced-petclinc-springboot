pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME = "jenkins-project-pet"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ACR_NAME = "azurejenkins"
        TENANT_ID = "d4f49458-0372-446d-aac3-fa4bf14ff177"
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        AKS_NAME = "jenkinsaks"
        RESOURCE_GROUP = "rg"
        K8S_DEPLOYMENT = "enhanced-petclinic"
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
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=sharmapalani_jenkins-project-pet \
                    -Dsonar.projectName=jenkins_project_pet \
                    -Dsonar.organization=sharmapalani \
                    -Dsonar.java.binaries=. \
                    '''
                }
            }
        }
        stage('maven package') {
            steps {
                echo "maven package"
                sh 'mvn package'
            }
        }
        stage('docker build') {
            steps {
                script{
                echo "docker build"
                docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        stage('ACR login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr_login', passwordVariable: 'AZURE_PASSWORD', usernameVariable: 'AZURE_USERNAME')]) {   
                script{
                sh '''
                echo "ACR login"
                az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                az acr login --name $ACR_NAME
                '''
                }
            }
        }
        }
        stage('docker push') {
            steps {
                script{
                echo "docker push"
                sh '''
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} $FULL_IMAGE_NAME
                docker push $FULL_IMAGE_NAME
                '''
                }
            }
        }
        stage('AKS login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr_login', passwordVariable: 'AZURE_PASSWORD', usernameVariable: 'AZURE_USERNAME')]) {
                script{
                sh '''
                echo "AKS login"
                az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME --overwrite-existing
                '''
                }
            }
        }
        }
        stage('deploy to AKS'){
            steps{
                script{
                    echo "deploy to AKS"
                    def output = sh(
                        script: "kubectl get deployment ${K8S_DEPLOYMENT} --ignore-not-found",returnStdout: true
                        ).trim()
                    def deploymentExists = output != ""
                    if (deploymentExists) {
                        echo "Deployment already exists. Updating..."
                        sh '''
                        kubectl set image deployment/${K8S_DEPLOYMENT} ${K8S_DEPLOYMENT}=${FULL_IMAGE_NAME}
                        '''
                    } else {
                        echo "Deployment does not exist. Creating..."
                        sh '''
                        sed "s/_IMAGE_TAG_/${IMAGE_TAG}/" k8s/deployment.yaml > k8s/tmp/deployment.yaml
                        kubectl apply -f k8s/tmp/deployment.yaml
                        '''
                    }
                }
            }
        }
    }
}