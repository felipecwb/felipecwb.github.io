pipeline {
    environment {
        dockerManagerHost = "${env.DOCKER_MANAGER_HOST}"
        DOMAIN = "${env.DOMAIN}"
        IMAGE = "felipecwb/site"
        stack = "site"
    }

    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build(IMAGE + ":build.$BUILD_NUMBER")
                }
            }
        }

        stage('Deploy Image') {
            when {
                branch 'master'
            }

            steps {
                script {
                    docker.withRegistry('', 'dockerhub') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy on Swarm') {
            when {
                branch 'master'
            }

            steps {
                sh '''
                    export DOCKER_HOST=${dockerManagerHost}
                    export ENVIRONMENT=production
                    export DOMAIN=${DOMAIN}

                    docker stack deploy \
                        --with-registry-auth \
                        --compose-file .docker/stack.yml \
                        ${stack}
                '''
            }
        }

        stage('Clean Workspace') {
            steps{
                echo 'Exited containers'
                sh 'docker container prune -f'

                echo 'images not attached'
                sh 'docker image prune -af'
            }
        }
    }
}
