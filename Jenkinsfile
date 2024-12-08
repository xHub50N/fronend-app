pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SSH_LOCAL_KEY_PATH = "/var/lib/jenkins/.ssh/githubWithoutPwd"
        TARGET_IP = "192.168.68.150"
        FOLDER_PATH = "/var/lib/jenkins/workspace/Cupid"
        DATA_FOLDER_PATH = "data"
        GIT_REPO_URL = "git@github.com:xHub50N/fronend-app.git"
        BRANCH_NAME = "main"
        REPO_DIR = "fronend-app"
    }

    stages {
        stage('Setup SSH Key') {
            steps {
                script {
                    if (!fileExists(env.SSH_LOCAL_KEY_PATH)) {
                        error "SSH key not found at: ${env.SSH_LOCAL_KEY_PATH}"
                    }

                    sh """
                        eval \$(ssh-agent -s)
                        ssh-add ${env.SSH_LOCAL_KEY_PATH}
                        ssh-keyscan -H github.com >> ~/.ssh/known_hosts
                    """
                }
            }
        }

        stage('Prepare Folder') {
            steps {
                script {
                    sh """
                        if [ ! -d "${env.FOLDER_PATH}" ]; then
                            echo "Creating folder: ${env.FOLDER_PATH}"
                            sudo mkdir -p ${env.FOLDER_PATH}
                            sudo chown jenkins:jenkins ${env.FOLDER_PATH}
                            sudo chmod 755 ${env.FOLDER_PATH}
                        else
                            echo "Folder already exists: ${env.FOLDER_PATH}"
                        fi
                    """
                }
            }
        }

       stage('Clone or Update Repository') {
           steps {
               dir("${env.FOLDER_PATH}") {
                   script {
                       sh """
                           pwd
                           if [ -d "${env.REPO_DIR}" ]; then
                               echo "Repository exists. Pulling latest changes..."
                               cd ${env.REPO_DIR}
                               git fetch origin
                               git checkout ${env.BRANCH_NAME}
                               git pull origin ${env.BRANCH_NAME}
                           else
                               echo "Cloning repository..."
                               git clone ${env.GIT_REPO_URL}
                               cd ${env.REPO_DIR}
                           fi
                       """
                   }
               }
           }
       }

       stage('Create Data Folders') {
           steps {
               dir("${env.FOLDER_PATH}") {
                   script {
                       sh """
                           echo "Creating 'data' directory with subfolders..."
                           sudo mkdir -p ${env.DATA_FOLDER_PATH}/backend/zdjecia ${env.DATA_FOLDER_PATH}/database
                           echo "Changing permissions for backend and database folders"
                           sudo chmod -R 777 ${env.DATA_FOLDER_PATH}/backend
                           sudo chown -R 10001:10001 ${env.DATA_FOLDER_PATH}/database
                       """
                   }
               }
           }
       }

        stage('Update Docker Compose File') {
           steps {
               dir("${env.FOLDER_PATH}/${env.REPO_DIR}") {
                   script {
                       sh """
                           echo "Replacing <your-ip-address> with ${env.TARGET_IP} in docker-compose.yaml"
                           sed -i "s/<your-ip-address>/${env.TARGET_IP}/g" docker-compose.yaml
                           sed -i "s|{URL}|http://${env.TARGET_IP}:8080/api|g" docker-compose.yaml
                       """
                    }
                }
            }
        }

        stage('Start Docker Containers') {
           steps {
               dir("${env.FOLDER_PATH}/${env.REPO_DIR}") {
                   script {
                       sh """
                           echo "Starting Docker containers..."
                           sudo docker compose up --build --force-recreate -d
                       """
                   }
               }
           }
       }
   }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}