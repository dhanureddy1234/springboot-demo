
pipeline {
    agent any

    environment {
        EC2_HOST = '3.106.205.187'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature/springboot-deployment',
                    url: 'https://github.com/dhanureddy1234/springboot-demo.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy Docker') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    ),
                    file(
                        credentialsId: 'demo1-db-env',
                        variable: 'DB_ENV_FILE'
                    )
                ]) {
                    bat """
                    scp -i "%SSH_KEY%" -o StrictHostKeyChecking=no target\\demo1-0.0.1-SNAPSHOT.jar %SSH_USER%@%EC2_HOST%:/opt/demo1/
                    scp -i "%SSH_KEY%" -o StrictHostKeyChecking=no Dockerfile %SSH_USER%@%EC2_HOST%:/opt/demo1/
                    scp -i "%SSH_KEY%" -o StrictHostKeyChecking=no "%DB_ENV_FILE%" %SSH_USER%@%EC2_HOST%:/opt/demo1/demo1-db.env

                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%EC2_HOST% "chmod 600 /opt/demo1/demo1-db.env"

                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%EC2_HOST% "sudo docker build -t demo1-app:%BUILD_NUMBER% /opt/demo1"

                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%EC2_HOST% "sudo docker rm -f demo1-container || true"

                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%EC2_HOST% "sudo docker run -d --restart unless-stopped --name demo1-container --add-host=host.docker.internal:host-gateway -p 8081:8080 --env-file /opt/demo1/demo1-db.env -e SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/myapp demo1-app:%BUILD_NUMBER%"
                    """
                }
            }
        }
    }
}
