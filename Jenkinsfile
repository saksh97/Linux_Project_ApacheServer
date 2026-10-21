pipeline {
    agent { label 'agent1' }

    parameters {
        // Dropdown menu allows you to switch target web deployments seamlessly
        choice(
            name: 'CHOSEN_WEBSITE', 
            choices: ['2117_infinite_loop', 'Zay_eCommerce', 'templatemo_596_electric_xtra'], 
            description: 'Select the website archive you want to deploy to your server.'
        )
    }

    environment {
        REPO_URL = 'https://github.com/thawaresameer715-cyber/Linux_Project_ApacheServer.git'
    }

    stages {
        stage('Verify Core Tools') {
            steps {
                echo 'Ensuring system tools (git and unzip) are available on the runner...'
                sh '''
                    sudo apt-get update -qq
                    sudo apt-get install -y git unzip
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                echo 'Cleaning up existing workspace directories...'
                cleanWs()
                echo "Fetching repository source files from: ${env.REPO_URL}"
                git branch: 'main', url: "${env.REPO_URL}"
            }
        }

        stage('Extract Selected Web Archive') {
            steps {
                script {
                    // Match the dropdown choice selection to the actual file naming system inside the repository
                    def zipFileName = ""
                    
                    if (params.CHOSEN_WEBSITE == '2117_infinite_loop') {
                        zipFileName = "2117_infinite_loop.zip"
                    } else if (params.CHOSEN_WEBSITE == 'Zay_eCommerce') {
                        zipFileName = "Zay eCommerce website files.zip"
                    } else if (params.CHOSEN_WEBSITE == 'templatemo_596_electric_xtra') {
                        zipFileName = "templatemo_596_electric_xtra.zip"
                    }
                    
                    echo "Extracting: ${zipFileName}"
                    // Extract the selected zip file package into a clean target directory
                    sh "unzip -o \"${zipFileName}\" -d target_extracted"
                }
            }
        }

        stage('Deploy Live to Web Server') {
            steps {
                echo "Deploying ${params.CHOSEN_WEBSITE} files to the server root..."
                sh """
                    # 1. Purge the existing active folder tree
                    sudo rm -rf /var/www/html/*
                    
                    # 2. Extract contents and dynamically handle nested folders inside the archive
                    # Some zips might contain spaces or distinct subfolder paths when unpacked
                    if [ -d "target_extracted/${params.CHOSEN_WEBSITE}" ]; then
                        sudo cp -r "target_extracted/${params.CHOSEN_WEBSITE}/"* /var/www/html/
                    elif [ -d "target_extracted/Zay eCommerce website files" ]; then
                        sudo cp -r "target_extracted/Zay eCommerce website files/"* /var/www/html/
                    else
                        sudo cp -r target_extracted/* /var/www/html/
                    fi
                    
                    # 3. Align system security ownership and read permissions
                    sudo chown -R www-data:www-data /var/www/html
                    sudo chmod -R 755 /var/www/html
                """
                echo 'Deployment execution complete!'
            }
        }
