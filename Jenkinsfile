########
# Jenkinsfile
# Writing a Jenkinsfile for a simple pipeline that builds and tests a Java application using Maven.
# It should include stages for checkout, build, test and push to a remote repository.
# For the application - Java version is 17 , for jenkins - Java version is 21.
# It should include agent ,triggers, environment variables, and post actions for cleanup.
# It should include a stage for code quality analysis using SonarQube.
# It should generate a report for the build and test results.
# Build and test should be done in a single stage.
# It should include a stage for docker login.
# After docker login, it should build a docker image for the application and push it to Docker Hub in a single stage.
# It should generate both an artifact and a Docker image for the application and push Docker image to Docker Hub.
# Image name: nidhi2425/jenkins_project_1:$BUILD_NUMBER
# git url: https://github.com/learnandgrow459-ui/Jenkins_project_1.git , branch: main
# Want to run CI pipeline on every commit to the main branch and want to run on a worker node with label "java-builder". Pollscm for 1 min for every commit to the main branch.
########



pipeline {
    agent { label 'java-builder' }
    triggers {
        pollSCM('H/1 * * * *') // Poll SCM every minute
    }
    environment {
        DOCKER_IMAGE = "nidhi2425/jenkins_project_1:${env.BUILD_NUMBER}"
        SONARQUBE_URL = "http://your-sonarqube-server"
        SONARQUBE_TOKEN = credentials('sonarqube-token') // Assuming you have a Jenkins credential for SonarQube token
        JAVA_HOME = tool name: 'JDK 17', type: 'jdk' // Assuming you have a JDK 17 configured in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/learnandgrow459-ui/Jenkins_project_1.git'
            }
        } 
        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Assuming you have a SonarQube server configured in Jenkins
                    sh 'mvn sonar:sonar -Dsonar.projectKey=jenkins_project_1 -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN'
                }
            }
        }
        stage('Build and Test') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        stage('Generate Test Report') {
            steps {
                publishHTML(target: [
                    reportName: 'Test Report',
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'index.html',
                    keepAll: true,
                    alwaysLinkToLastBuild: true,
                    allowMissing: false
                ])
            }
        }
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
                sh 'docker push $DOCKER_IMAGE'
            }
        }
        stage('Cleanup') {
            steps {
                sh 'docker logout'
                cleanWs()
            }
        }
        stage('Post Actions') {
            steps {
                echo 'Pipeline completed....'
            }
        }

