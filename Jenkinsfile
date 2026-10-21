pipeline {
    agent { label 'agent1' } 

    parameters {
        choice(
            name: 'CHOSEN_WEBSITE', 
            choices: ['2117_infinite_loop', 'Zay_eCommerce', 'templatemo_596_electric_xtra'], 
            description: 'Select the website archive you want to deploy to the Nginx container.'
        )
    }

    environment {
        REPO_URL = 'https://github.com/saksh97/Linux_Project_ApacheServer.git'
        NGINX_CONTAINER_NAME = 'my-nginx-container'
    }

    stages {
        stage('Verify & Install Docker Engine') {
            steps {
                echo 'Ensuring system tools and docker engine are operational...'
                sh '''
                    sudo apt-get update -qq
                    sudo apt-get install -y unzip docker.io
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                echo 'Cleaning workspace and fetching repository updates...'
                cleanWs()
                git branch: 'main', url: "${env.REPO_URL}"
            }
        }

        stage('Extract Web Assets') {
            steps {
                script {
                    def zipMap = [
                        '2117_infinite_loop': '2117_infinite_loop.zip',
                        'Zay_eCommerce': 'Zay eCommerce website files.zip',
                        'templatemo_596_electric_xtra': 'templatemo_596_electric_xtra.zip'
                    ]
                    
                    def exactZipFile = zipMap[params.CHOSEN_WEBSITE]
                    echo "Extracting the zipped archive for: ${exactZipFile}..."
                    sh "unzip -o \"${exactZipFile}\" -d target_extracted"
                }
            }
        }

        stage('Deploy directly to Nginx Container') {
            steps {
                echo "Managing lifecycle and deploying to ${env.NGINX_CONTAINER_NAME}..."
                sh """
                    # 1. Check if the target Nginx container is already running
                    if [ \$(sudo docker ps -a -q -f name=^/${env.NGINX_CONTAINER_NAME}\$) ]; then
                        echo "Container exists. Ensuring it is actively running..."
                        sudo docker start ${env.NGINX_CONTAINER_NAME}
                    else
                        echo "Container does not exist. Spinning up a fresh Nginx container on port 80..."
                        sudo docker run -d -p 80:80 --name ${env.NGINX_CONTAINER_NAME} nginx:alpine
                        # Give the container a brief moment to initialize file pathways
                        sleep 3
                    fi

                    # 2. Clear default out-of-the-box landing page assets
                    sudo docker exec ${env.NGINX_CONTAINER_NAME} rm -rf /usr/share/nginx/html/*
                    
                    # 3. Inject the targeted extraction directory assets into the container filesystem
                    if [ -d "target_extracted/2117_infinite_loop" ]; then
                        sudo docker cp target_extracted/2117_infinite_loop/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    elif [ -d "target_extracted/Zay eCommerce website files" ]; then
                        sudo docker cp "target_extracted/Zay eCommerce website files/." ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    elif [ -d "target_extracted/templatemo_596_electric_xtra" ]; then
                        sudo docker cp target_extracted/templatemo_596_electric_xtra/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    else
                        sudo docker cp target_extracted/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    fi
                    
                    # 4. Gracefully refresh Nginx runtime configurations without container restarts
                    sudo docker exec ${env.NGINX_CONTAINER_NAME} nginx -s reload
                """
                echo 'Container hot-swap deployment completed successfully!'
            }
        }
    }
}
