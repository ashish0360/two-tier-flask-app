pipeline{
    
    agent {label "prd"}
    
    environment{
        DOCKER_IMAGE = "two-tier-flask-app"
    }

    stages{
        stage("cloing code"){
            steps{
                echo "Cloning the code"
                git url:"https://github.com/ashish0360/two-tier-flask-app.git", branch: "master"
            }
        }
        stage("buidling clode"){
            steps{
                echo "buidling the code"
                sh "dockre build -t two-tier-flask-app ."
            }
        }
        stage("pushing to docker hub"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerhubcred", passwordVariable:"dockerhubpass", usernameVariable:"dockerhubuser")]){
                sh "docker login -u ${env.dockerhubuser} -p ${env.dockerhubpass}"
                sh "docker image tag two-tier-flask-app ${env.dockerhubuser}/$DOCKER_IMAGE:latest"
                sh "docker push ${env.dockerhubuser}/$DOCKER_IMAGE:latest"
                }
            }
        } 
        stage("testing clode"){
            steps{
                echo "testing the code"
            }
        }
        stage("deploying code"){
            steps {
                echo "Deploying the code"
                sh '''
                    set -eu
                    docker rm -f mysql flask-app 2>/dev/null || true
                    docker compose -p two-tier-flask-app down --remove-orphans
                    docker compose -p two-tier-flask-app up -d --build --force-recreate
                    docker compose -p two-tier-flask-app ps
                '''
            }
        }
    }

    post{
        success{
            echo "pipeline built successfully"
            emailext (
                to: 'ashish2024.ak@gmail.com',
                subject: "Pipeline Built Successfully",
                body: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            )
        }
        failure{
            echo "pipeline broken"
            emailext (
                to: 'ashish2024.ak@gmail.com',
                subject: "Pipeline Broken",
                body: "BROKEN: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            )
        }
    }
}
