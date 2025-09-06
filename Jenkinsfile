pipeline {
    agent any

    environment {
        GIT_REPO    = 'https://github.com/jahnavinnusum/fullstack-s104.git'  
        TOMCAT_URL  = 'http://3.220.86.9:9090/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'adminadmin'
        PEM_FILE    = '/home/ec2-user/myec2key-.pem'
        EC2_IP      = '3.220.86.9'
        APP_NAME    = 'springapp1'  // Tomcat context path
        FRONTEND_DEST = '/var/www/html/'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Backend') {
            steps {
                dir('CRUDBack') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('CRUDFront') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy Backend to Tomcat') {
            steps {
                dir('CRUDBack/target') {
                    script {
                        WAR_FILE = sh(
                            script: "ls *.war | head -n 1",
                            returnStdout: true
                        ).trim()
                        
                        if (!WAR_FILE) {
                            error "No WAR file found! Build might have failed."
                        }

                        echo "Deploying $WAR_FILE to Tomcat..."
                        sh """
                            curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \
                            --upload-file $WAR_FILE \
                            "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"
                        """
                    }
                }
            }
        }

        stage('Deploy Frontend to EC2') {
            steps {
                dir('CRUDFront/build') {
                    sh """
                        echo "Cleaning old frontend files..."
                        ssh -o StrictHostKeyChecking=no -i ${PEM_FILE} ec2-user@${EC2_IP} "rm -rf ${FRONTEND_DEST}*"

                        echo "Copying new frontend build to EC2..."
                        scp -o StrictHostKeyChecking=no -i ${PEM_FILE} -r * ec2-user@${EC2_IP}:${FRONTEND_DEST}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
