pipeline {
    agent any

    // Adjust these options as needed
    options {
        timestamps()
        buildDiscarder(logRotator(daysToKeepStr: '14', numToKeepStr: '10'))
    }

    tools {
        // Assumes Jenkins global JDK installation named "JDK11"
        jdk 'JDK11'
        // If you're using Maven in your Jenkins tools config:
        maven 'Maven3'
    }

    environment {
        // Example environment variables
        MAVEN_OPTS = '-Xmx1024m'
        // Change as necessary, e.g. for Docker registry credentials
        // DOCKER_REGISTRY = 'myregistry.example.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: env.GIT_BRANCH ?: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/bharat2905-cpu/banking-API.git']]
                ])
            }
        }

        stage('Build & Test') {
            steps {
                // If using the Maven wrapper (./mvnw), replace with that
                sh 'mvn clean verify -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/**/*.xml'
                    // Adjust test report path if using other reporters
                }
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Static Code Analysis') {
            when {
                expression { fileExists('pom.xml') }
            }
            steps {
                // Example: run SpotBugs or other analysis; wrap appropriately
                sh 'mvn spotbugs:check -B || true'
                recordIssues tools: [spotBugs(pattern: '**/target/spotbugsXml.xml')]
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B'
            }
        }

        stage('Publish Artifact') {
            when {
                branch 'main'
            }
            steps {
                // Example: deploy to Nexus or Artifactory
                // sh 'mvn deploy -DskipTests -B'
                echo 'Deploying artifact to repository (configure in Jenkins credentials)'
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def image = "myrepo/banking-api:${env.BUILD_NUMBER}"
                    sh "docker build -t ${image} ."
                    // Authenticate to your Docker registry
                    // sh "docker login -u user -p pass ${env.DOCKER_REGISTRY}"
                    sh "docker push ${image}"
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo 'Trigger deployment (e.g., kubectl apply, helm upgrade)'
            }
        }
    }

    post {
        failure {
            mail to: 'dev-team@example.com',
                 subject: "FAILED: Job '${env.JOB_NAME} [#${env.BUILD_NUMBER}]'",
                 body: "Something went wrong with build ${env.BUILD_URL}"
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
