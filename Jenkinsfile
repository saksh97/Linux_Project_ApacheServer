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
        stage('Verify Host Tools') {
            steps {
                echo 'Ensuring system tools are available on the runner host...'
                sh 'sudo apt-get update -qq && sudo apt-get install -y unzip'
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
                    // This map pairs your dropdown choice with the exact filename in GitHub
                    def zipMap = [
                        '2117_infinite_loop': '2117_infinite_loop.zip',
                        'Zay_eCommerce': 'Zay eCommerce website files.zip',
                        'templatemo_596_electric_xtra': 'templatemo_596_electric_xtra.zip'
                    ]
                    
                    def exactZipFile = zipMap[params.CHOSEN_WEBSITE]
                    echo "Extracting the zipped archive for: ${exactZipFile}..."
                    
                    // Uses double quotes to handle filenames with spaces correctly
                    sh "unzip -o \"${exactZipFile}\" -d target_extracted"
                }
            }
        }

        stage('Deploy directly to Nginx Container') {
            steps {
                echo "Injecting static website code directly into ${env.NGINX_CONTAINER_NAME}..."
                sh """
                    # 1. Wipe the default assets inside the running Nginx container
                    docker exec ${env.NGINX_CONTAINER_NAME} rm -rf /usr/share/nginx/html/*
                    
                    # 2. Deploy extracted workspace directory directly inside the container filesystem
                    if [ -d "target_extracted/2117_infinite_loop" ]; then
                        docker cp target_extracted/2117_infinite_loop/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    elif [ -d "target_extracted/Zay eCommerce website files" ]; then
                        docker cp "target_extracted/Zay eCommerce website files/." ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    elif [ -d "target_extracted/templatemo_596_electric_xtra" ]; then
                        docker cp target_extracted/templatemo_596_electric_xtra/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    else
                        docker cp target_extracted/. ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    fi
                    
                    # 3. Reload Nginx internally to refresh configuration seamlessly
                    docker exec ${env.NGINX_CONTAINER_NAME} nginx -s reload
                """
                echo 'Container hot-swap deployment successful!'
            }
        }
    }
}
