pipeline{
    agent any
    tools {
        jdk 'java17'
        maven 'maven3'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
                git 'https://github.com/mukeshr-29/devops-project-2.2.git'
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage("quality check using sonarqube"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube'){
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage("quality gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId:'sonarqube'
                }
            }
        }
        stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('ansible playbook docker'){
            steps{
                script{
                    ansiblePlaybook become: true, credentialsId: 'ansible', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                }
            } 
        }
        stage('k8s using ansible'){
            steps{
                script{
                    echo 'Current Workspace: ' + env.WORKSPACE
                    ansiblePlaybook become: true, credentialsId: 'ansible', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                
                }
            } 
        }
    }

}
