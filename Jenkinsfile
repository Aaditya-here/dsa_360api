pipeline {
    agent any
    
    environment {
        DATE_TIME = new Date().format('yyyyMMddHHmmss')
        DOCKER_IMAGE_NAME = "aadityapatil08/dsa360-${DATE_TIME}"
        DOCKERFILE_PATH = 'Dockerfile'
        DOCKERHUB_CREDENTIALS = credentials('dockerpassword')
        PREVIOUS_IMAGE = ''
    }

    tools {
        maven "MAVEN_HOME"
    }

    stages {
        stage('Get Code') {
            steps {
                echo "Cloning the repository..."
                git branch: 'main', url: 'https://github.com/Aaditya-here/dsa_360api.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building the Maven project..."
                bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                withSonarQubeEnv('MySonarQubeServer') {
                    bat "mvn sonar:sonar"
                }
            }
        }

        stage('Find Previous Docker Image') {
            steps {
                script {
                    echo "Searching for previous Docker image..."

                    // List all Docker images and filter programmatically
                    def existingImages = bat(script: 'docker images --format "{{.Repository}}:{{.Tag}}"', returnStdout: true).trim()

                    PREVIOUS_IMAGE = ''
                    existingImages.split('\n').each { img ->
                        if (img.contains("aadityapatil08/dsa360-")) {
                            PREVIOUS_IMAGE = img.trim()
                            echo "Found previous image: ${PREVIOUS_IMAGE}"
                        }
                    }

                    if (PREVIOUS_IMAGE) {
                        echo "Previous image found: ${PREVIOUS_IMAGE}"
                    } else {
                        echo "No previous image found. Proceeding with a new build."
                    }
                }
            }
        }

        stage('Remove Previous Docker Image') {
            when {
                expression { PREVIOUS_IMAGE != '' }
            }
            steps {
                script {
                    echo "Removing previous Docker image: ${PREVIOUS_IMAGE}"
                    bat "docker rmi ${PREVIOUS_IMAGE} || echo 'Failed to delete previous image or not found.'"
                }
            }
        }

        stage('Build New Docker Image') {
            steps {
                script {
                    echo "Building new Docker image..."
                    def dockerImage = docker.build(env.DOCKER_IMAGE_NAME, "--cache-from ${env.DOCKER_IMAGE_NAME} -f ${env.DOCKERFILE_PATH} .")
                    echo "Docker image built: ${env.DOCKER_IMAGE_NAME}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to DockerHub..."
                    withCredentials([string(credentialsId: 'dockertoken', variable: 'dockertoken')]) {
                        bat "docker login -u aadityapatil08 -p ${dockertoken}"
                        bat "docker push ${env.DOCKER_IMAGE_NAME}"
                        echo "Docker image pushed successfully!"
                    }
                }
            }
        }

        stage('Cleanup Workspace') {
            steps {
                echo "Cleaning up unnecessary files..."
                bat "mvn clean"
                bat "docker system prune -f"
            }
        }
    }

    post {
    success {
        script {
            echo '✅ Pipeline succeeded! Sending success email...'
            emailext (
                to: 'aadityapatilpush@gmail.com',  // Replace with your email
                subject: "✅ Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>Jenkins Build Successful 🎉</h2>
                    <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Docker Image Pushed:</strong> ${env.DOCKER_IMAGE_NAME}</p>
                    <p>Check the full logs in the attachment.</p>
                    <p><a href="${env.BUILD_URL}console">View Console Logs</a></p>
                """,
                attachLog: true,
                mimeType: 'text/html'
            )
        }
    }
    
    failure {
        script {
            echo '❌ Pipeline failed! Sending failure email...'
            emailext (
                to: 'aadityapatilpush@gmail.com',  // Replace with your email
                subject: "❌ Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>Jenkins Build Failed ❌</h2>
                    <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Failed Stage:</strong> ${currentBuild.currentResult}</p>
                    <p>Check the attached logs for details.</p>
                    <p><a href="${env.BUILD_URL}console">View Console Logs</a></p>
                """,
                attachLog: true,
                mimeType: 'text/html'
            )
        }
    }
}

}
