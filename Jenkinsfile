pipeline {
    agent any

    tools {
        maven 'maven-3.9.6'
    }

    environment {
        // Segregated application name to coexist cleanly alongside your previous application
        APP_NAME         = 'spring-boot-gitops-app'
        DOCKER_USER      = 'indu1999'
        IMAGE_TAG        = "${DOCKER_USER}/${APP_NAME}:${BUILD_NUMBER}"
        
        // Coordinates for your dedicated infrastructure manifest repository
        GITOPS_REPO_NAME = 'spring-boot-gitops'
        GITOPS_REPO_URL  = "github.com/Induwara1999/${GITOPS_REPO_NAME}.git"
    }

    stages {
        stage('1. Maven Build & Test') {
            steps {
                echo 'Compiling App Code using Native Maven Tool...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('2. Build & Push Docker Image') {
            steps {
                echo "Installing Docker CLI binary and managing image lifecycle..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'HUB_USER', passwordVariable: 'HUB_TOKEN')]) {
                    sh """
                        # Pull local static Docker binary if missing
                        if [ ! -f ./docker ]; then
                            echo "Downloading Docker CLI v26.1.4..."
                            curl -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-26.1.4.tgz | tar -xzO docker/docker > ./docker
                            chmod +x ./docker
                        fi
                        
                        # Build container image matching the new application tag path
                        ./docker build -t ${IMAGE_TAG} .
                        
                        # Authenticate and upload to Docker Hub
                        echo "\$HUB_TOKEN" | ./docker login --username "${DOCKER_USER}" --password-stdin
                        ./docker push ${IMAGE_TAG}
                    """
                }
            }
        }

        stage('3. Update GitOps Manifests Repository') {
            steps {
                echo "Modifying deployment tracking files inside declarative manifest structures..."
                // Successfully binds using your exact 'github-credentials' ID mapping
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')]) {
                    sh """
                        # Clean up leftover workspaces from older system check iterations
                        rm -rf ${GITOPS_REPO_NAME}
                        
                        # Set up local workspace Git context identity for automation tracing
                        git config --global user.email "jenkins-automation@local.infra"
                        git config --global user.name "Jenkins GitOps Engine"
                        
                        # Authenticate securely using the Personal Access Token and clone the Manifest Repo
                        git clone https://${GH_TOKEN}@${GITOPS_REPO_URL}
                        cd ${GITOPS_REPO_NAME}
                        
                        # Find the image field inside deployment.yaml and update it to the current build number
                        sed -i "s|image: ${DOCKER_USER}/${APP_NAME}:.*|image: ${IMAGE_TAG}|g" deployment.yaml
                        
                        # Stage, commit, and push changes back up to GitHub for ArgoCD to detect
                        git add deployment.yaml
                        git commit -m "automated-ops: update image tracker flag target to release v${BUILD_NUMBER}"
                        git push origin main
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
