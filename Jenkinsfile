pipeline {
    agent any
    triggers {
        pollSCM "* * * * *"
    }
    stages {
        stage('Initialize') {
            steps {
                script {
                    // Getting the latest commit hash and shortening it
                    GIT_COMMIT_HASH = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    SHORT_COMMIT = GIT_COMMIT_HASH.take(7)
                }
            }
        }
        stage('Build Application') {
            steps {
                echo '=== Building Petclinic Application ==='
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test Application') {
            steps {
                echo '=== Testing Petclinic Application ==='
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                echo '=== Building Petclinic Docker Image ==='
                script {
                    // Docker image build step using the defined SHORT_COMMIT for tagging
                    app = docker.build("juniemariam/amazon-eks-jenkins-terraform:${SHORT_COMMIT}")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                echo '=== Pushing Petclinic Docker Image ==='
                script {
                    // Pushing the Docker image with the SHORT_COMMIT and latest tags
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                        app.push("${SHORT_COMMIT}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                // Deleting the local Docker images to clean up space
                sh "docker rmi -f juniemariam/amazon-eks-jenkins-terraform:${SHORT_COMMIT} || :"
                sh "docker rmi -f juniemariam/amazon-eks-jenkins-terraform:latest || :"
            }
        }
    }
}
