pipeline {
    agent any

    parameters {
        // Git
        string(name: 'GIT_URL', defaultValue: 'git@github.com:HubblerMobile/hubbler-web-auth.git', description: 'Git repository URL')
        string(name: 'GIT_BRANCH', defaultValue: 'login-azure-dev', description: 'Git branch to clone')
        string(name: 'CREDENTIALS_ID', defaultValue: 'github_ssh', description: 'Jenkins credentials ID for Git authentication')
        // Ansible
        string(name: 'HOST_FILE', defaultValue: 'az/sand/workspace/ansible/hosts/az-sand-env', description: 'Ansible inventory file location within the repository')
        string(name: 'TARGET_GROUP', defaultValue: 'az-sand-web', description: 'Ansible group for target servers')
        //string(name: 'Deploy_Code_PLAYBOOK_PATH', defaultValue: 'FC/Dev/workspace/ansible/ansible_playbooks/Web/deploy_code.yml', description: 'Path to Ansible playbook within the repository')
        string(name: 'Deploy_Code_PLAYBOOK_PATH', defaultValue: 'az/sand/workspace/ansible/ansible_playbooks/web/module_wise_deploy_code.yml', description: 'Path to Ansible playbook within the repository')
        string(name: 'Stop_Services_PLAYBOOK_PATH', defaultValue: 'az/sand/workspace/ansible/ansible_playbooks/web/stop_service.yml', description: 'Path to Ansible playbook for stopping services')
        string(name: 'Start_Services_PLAYBOOK_PATH', defaultValue: 'az/sand/workspace/ansible/ansible_playbooks/web/start_service.yml', description: 'Path to Ansible playbook for starting services')

        // Email IDs
        string(name: 'EMAIL_LIST_FILE', defaultValue: 'az/sand/workspace/email_list/az_sand_env_email_list.txt', description: 'File containing email addresses for notification')
        // User
        string(name: 'REMOTE_USER', defaultValue: 'ansible', description: 'Username for SSH connection')
        // Dir
        //string(name: 'BASE_TARGET_DIR', defaultValue: '/home/RootRiteshR/fc_target_folder', description: 'Base target directory path')
        string(name: 'BASE_TARGET_DIR', defaultValue: '/var/www/html/', description: 'Base target directory path')

    }

    environment {
        SRC_FOLDER = "/var/lib/jenkins/workspace/${JOB_NAME}/source"
        SCRIPT_FOLDER = "/var/lib/jenkins/workspace/${JOB_NAME}/pipeline_scripts"
        SUB_FOLDER = "login"  // Variable to specify the subfolder
        //TARGET_DIR = "${BASE_TARGET_DIR}/${params.GIT_URL.tokenize('/')[-1].replace('.git', '')}/${params.GIT_BRANCH}"
        TARGET_DIR = "${BASE_TARGET_DIR}/${env.SUB_FOLDER}"
       

        // Scripts
        PIPELINE_SCRIPT_REPO = 'git@github.com:HubblerMobile/devops.git'
        PIPELINE_SCRIPT_BRANCH = 'azure-sand-env-jenkins'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                sh 'rm -rf $SRC_FOLDER $SCRIPT_FOLDER'
            }
        }

        stage('Checkout Pipeline Scripts') {
            steps {
                echo 'Checking out the pipeline scripts and data...'
                script {
                    // Create directories if not available
                    sh 'mkdir -p $SCRIPT_FOLDER'
                    sh 'mkdir -p $SRC_FOLDER'
                }
                retry(3) {
                    echo 'Checking out the Git repository for pipeline scripts...'
                    checkout([$class: 'GitSCM', branches: [[name: env.PIPELINE_SCRIPT_BRANCH]], userRemoteConfigs: [[credentialsId: env.CREDENTIALS_ID, url: env.PIPELINE_SCRIPT_REPO]], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'pipeline_scripts']]])
                }
            }
        }

        stage('Checkout Source Code') {
            steps {
                echo 'Checking out the Git repository for source code...'
                retry(3) {
                    checkout([$class: 'GitSCM', branches: [[name: params.GIT_BRANCH]], userRemoteConfigs: [[credentialsId: params.CREDENTIALS_ID, url: params.GIT_URL]], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'source']]])
                }
            }
        }

        stage('Prepare') {
            steps {
                echo 'Preparing the pipeline...'
                // List files in the source directory
                echo 'Files in the source directory:'
                sh 'ls -la $SRC_FOLDER'

                // List files in the pipeline scripts directory
                echo 'Files in the pipeline scripts directory:'
                sh 'ls -la $SCRIPT_FOLDER'
            }
        }



        //

    }

post {
    always {
        script {
            echo 'Reading email addresses...'
            try {
                env.EMAIL_LIST = readFile("${SCRIPT_FOLDER}/${params.EMAIL_LIST_FILE}").trim()
            } catch (Exception e) {
                echo 'Error: Unable to read email addresses.'
            }

            echo 'Sending email notification...'

            mail(
                to: env.EMAIL_LIST,
                subject: "Pipeline ${currentBuild.result}: ${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]",
                body: """
                    <html>
                        <body>
                            <p>Hello Team,</p>
                            <p>Pipeline ${env.JOB_NAME} completed with status: ${currentBuild.result}.</p>
                            <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        </body>
                    </html>
                """,
                mimeType: 'text/html',
                from: 'your-gmail-address@gmail.com',
                smtpServer: 'smtp.gmail.com',
                smtpPort: 587,
                useSSL: true,
                auth: true,
                userName: 'your-gmail-address@gmail.com',
                password: 'your-gmail-password',
                attachLog: true,
                attachmentsPattern: '**/console.log' // Attach console log file
            )

            echo 'Listing contents of the workspace directory before cleanup:'
            sh 'ls -la /var/lib/jenkins/workspace/${JOB_NAME}'

            echo 'Cleaning up the workspace directory...'
            sh 'rm -rf /var/lib/jenkins/workspace/${JOB_NAME}/*'

            echo 'Listing contents of the workspace directory after cleanup:'
            sh 'ls -la /var/lib/jenkins/workspace/${JOB_NAME}'

            echo 'Cleanup complete.'
        }
    }
}

}
