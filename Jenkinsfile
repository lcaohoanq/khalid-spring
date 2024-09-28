pipeline {
    agent any

    tools { 
        maven 'my-maven' 
    }
    
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
        DOCKER_CLI_VERSION = '17.04.0-ce'
    }
    
    stages {
        stage('Install Docker CLI') {
            steps {
                script {
                    sh """
                        curl -fsSLO https://get.docker.com/builds/Linux/x86_64/docker-${DOCKER_CLI_VERSION}.tgz
                        tar xzvf docker-${DOCKER_CLI_VERSION}.tgz
                        mv docker/docker /usr/local/bin
                        rm -r docker docker-${DOCKER_CLI_VERSION}.tgz
                        docker --version
                    """
                }
            }
        }
        
        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop lcaohoanq-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm lcaohoanq-mysql-data || echo "no volume"'

                sh "docker run --name lcaohoanq-mysql --rm --network dev -v lcaohoanq-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i lcaohoanq-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t lcaohoanq/springboot .'
                    sh 'docker push lcaohoanq/springboot'
                }
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull lcaohoanq/springboot'
                sh 'docker container stop lcaohoanq-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name lcaohoanq-springboot -p 8081:8080 --network dev lcaohoanq/springboot'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}