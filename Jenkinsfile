pipeline {
    agent any

    stages {
        stage('Stage 1: Start FastAPI Service') {
            steps {
                script {
                    // Clone the repository
                    git branch: 'main', url: 'https://github.com/hocinilotfi/fastapi-jenkins'
                    
                    // Create a custom Docker network (if it doesn't exist)
                   sh 'docker network create jenkins || true'
                    
                    // Start Docker Compose in detached mode with the custom network
                     sh 'docker ps'
                   sh 'docker stop 8bad87db8812'
                    sh 'docker compose down'
                   
                    sh 'docker compose up -d'

                    // Wait for the FastAPI service to be up
                    sh """
                    for i in {1..10}; do
                        if curl --silent --fail http://localhost:8988; then
                            echo 'FastAPI service is up and running.'
                            break
                        else
                            echo "FastAPI service is not available yet. Retrying in 5 seconds..."
                            sleep 5
                        fi
                    done
                    """

                    // List running containers
                    sh 'docker ps'
                }
            }
        }

        stage('Stage 2: Get Newman Image') {
            agent {
                docker { 
                    image 'postman/newman'
                    args "--network=jenkins --entrypoint=''"
                }
            }

            steps {
                script {
                    // Clone the Newman collection repository
                    git branch: 'main', url: 'https://github.com/cabrelken/newman-collections'
                    
                    // Run Newman with the collection
                    sh 'newman -v'
                    sh 'newman run collection1.json -e jenkins.postman_environment.json'
                }
            }
        }
    }

    post {
        always {
            // Tear down the Docker Compose setup at the end
            sh "docker compose down"
           
        }
    }
}
