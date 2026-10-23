
pipeline {

    // Select the Jenkins agent/node labelled "prd" to execute this pipeline.
    agent { label "prd" }

    // ALTERNATIVE:
    // Use "agent any" if the pipeline can run on any available Jenkins agent.
    // Do not use both agent declarations at the same time.
    // agent any;

    environment {
        // Define a variable to store the Docker image name.
        // This variable can be reused in different pipeline stages.
        DOCKER_IMAGE = "two-tier-flask-app"
    }

    stages {

        // STAGE 1: CLONE CODE
        // Download the application source code from the GitHub repository.
        stage("cloning code") {
            steps {
                echo "Cloning the code"

                // Clone the master branch into the Jenkins workspace.
                git url: "https://github.com/ashish0360/two-tier-flask-app.git",
                    branch: "master"
            }
        }

        // STAGE 2: BUILD DOCKER IMAGE
        // Create a Docker image using the Dockerfile in the current directory.
        stage("building code") {
            steps {
                echo "Building the code"

                // -t assigns a name/tag to the image.
                // The dot (.) means the current directory is the build context.
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        // STAGE 3: PUSH IMAGE TO DOCKER HUB
        // Authenticate with Docker Hub, tag the image, and upload it.
        stage("pushing to Docker Hub") {
            steps {

                // Fetch the Docker Hub username and password from Jenkins Credentials.
                // "dockerhubcred" is the ID of the credentials saved in Jenkins.
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerhubcred",
                        passwordVariable: "dockerhubpass",
                        usernameVariable: "dockerhubuser"
                    )
                ]) {

                    // Log in securely without putting the password directly in the command.
                    sh 'echo "$dockerhubpass" | docker login -u "$dockerhubuser" --password-stdin'

                    // Tag the local image with the Docker Hub username and repository.
                    // Format: username/image-name:tag
                    sh 'docker image tag "$DOCKER_IMAGE" "$dockerhubuser/$DOCKER_IMAGE:latest"'

                    // Upload the tagged image to Docker Hub.
                    sh 'docker push "$dockerhubuser/$DOCKER_IMAGE:latest"'
                }
            }
        }

        // STAGE 4: TEST CODE
        // This currently prints a message; add actual automated tests here later.
        stage("testing code") {
            steps {
                echo "Testing the code"
            }
        }

        // STAGE 5: DEPLOY APPLICATION
        // Use Docker Compose to recreate and start the application containers.
        stage("deploying code") {
            steps {
                echo "Deploying the code"

                sh '''
                    # Stop if a command fails or an undefined variable is used.
                    set -eu

                    # Remove existing MySQL and Flask containers if they exist.
                    # "|| true" prevents failure if these containers are absent.
                    docker rm -f mysql flask-app 2>/dev/null || true

                    # Stop and remove containers belonging to this Compose project.
                    # --remove-orphans removes leftover containers from the project.
                    docker compose -p two-tier-flask-app down --remove-orphans

                    # Build images if needed, recreate containers, and run in the background.
                    # -p sets the Compose project name.
                    # -d runs containers in detached mode.
                    # --build builds images before starting.
                    # --force-recreate recreates containers even if their configuration is unchanged.
                    docker compose -p two-tier-flask-app up -d --build --force-recreate

                    # Display the status of the Compose-managed containers.
                    docker compose -p two-tier-flask-app ps
                '''
            }
        }
    }

    // POST ACTIONS
    // These actions run after the pipeline finishes.

    post {

        // Run this block if the pipeline completes successfully.
        success {
            echo "Pipeline built successfully"

            // Email a success notification with the Jenkins job and build number.
            emailext(
                to: 'ashish2024.ak@gmail.com',
                subject: "Pipeline Built Successfully",
                body: """
                <h2>Pipeline Successful</h2>
                <p><b>Triggered By:</b> ${env.BUILD_USER}</p>
                <p><b>Job Name:</b> ${env.JOB_NAME}</p>
                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }

        // Run this block if the pipeline fails.
        failure {
            echo "Pipeline broken"

            // Email a failure notification with the Jenkins job and build number.
            emailext(
                to: 'ashish2024.ak@gmail.com',
                subject: "Pipeline Broken",
                body: """
                <h2>Pipeline Broken</h2>
                <p><b>Triggered By:</b> ${env.BUILD_USER}</p>
                <p><b>Job Name:</b> ${env.JOB_NAME}</p>
                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }
    }
}