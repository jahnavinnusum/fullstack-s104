pipeline {
    agent any

    environment {
        GIT_REPO    = 'https://github.com/your-username/your-repo.git'  // replace with your repo
        TOMCAT_URL  = 'http://3.220.86.9:9090/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'
        PEM_FILE    = '/home/ec2-user/myec2key-.pem'
        EC2_IP      = '3.220.86.9'
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
                    sh '''
                        WAR_FILE=$(ls *.war | head -n 1)
                        echo "Deploying $WAR_FILE to Tomcat..."
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file $WAR_FILE \
                        "${TOMCAT_URL}/deploy?path=/springapp1&update=true"
                    '''
                }
            }
        }

        stage('Deploy Frontend to EC2') {
            steps {
                dir('CRUDFront/build') {
                    sh '''
                        echo "Copying frontend build to EC2..."
                        scp -o StrictHostKeyChecking=no -i ${PEM_FILE} -r * ec2-user@${EC2_IP}:/var/www/html/
                    '''
                }
            }
        }
    }
}
