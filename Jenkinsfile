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
                    # 1. Stop and remove the old container completely to wipe cached layers
                    if [ \$(sudo docker ps -a -q -f name=^/${env.NGINX_CONTAINER_NAME}\$) ]; then
                        echo "Removing old container execution state to prevent caching..."
                        sudo docker rm -f ${env.NGINX_CONTAINER_NAME}
                    fi

                    # 2. Spin up a clean, isolated Nginx container instance on port 81
                    echo "Spinning up a fresh container instance..."
                    sudo docker run -d -p 81:80 --name ${env.NGINX_CONTAINER_NAME} nginx:alpine
                    sleep 3

                    # 3. Clear default container template landing assets
                    sudo docker exec ${env.NGINX_CONTAINER_NAME} rm -rf /usr/share/nginx/html/*
                    
                    # 4. Inject the selected extraction directory assets directly into the filesystem
                    if [ -d "target_extracted/2117_infinite_loop" ]; then
                        sudo docker cp target_extracted/2117_infinite_loop/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    elif [ -d "target_extracted/Zay eCommerce website files" ]; then
                        sudo docker cp "target_extracted/Zay eCommerce website files/." ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    elif [ -d "target_extracted/templatemo_596_electric_xtra" ]; then
                        sudo docker cp target_extracted/templatemo_596_electric_xtra/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    else
                        sudo docker cp target_extracted/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    fi
                    
                    # 5. Set proper permissions inside the container environment
                    sudo docker exec ${env.NGINX_CONTAINER_NAME} chmod -R 755 /usr/share/nginx/html
                """
                echo 'Container hot-swap deployment completed successfully!'
            }
        }
    }
}
