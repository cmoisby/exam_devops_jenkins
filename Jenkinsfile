pipeline {
    environment {  
        DOCKER_ID = "cmoisby"  
        DOCKER_IMAGE = "Jenkins_devops_exams"
        DOCKER_TAG = "v.${BUILD_ID}.0"  
    }
    agent any  // Jenkins will be able to select all available agents
    stages {

        stage("Datascientest Env Variables") {
            steps {
                echo "The build id is ${env.BUILD_ID} or ${BUILD_ID} or ${BUILD_ID} "
            }
        }

        stage('Docker Build') { // Docker build image stage
            steps {
                script {
                    sh '''
                        docker rm -f cast-service
                        docker rm -f movie-service
                        echo "${DOCKER_ID}/${DOCKER_IMAGE}"
                        docker build -t ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG} ./cast
                        docker build -t ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG} ./movie
                        sleep 6
                    '''
                }
            }
        }

        stage('Docker run') { // Run containers from our built images
            steps {
                script {
                    sh '''
                        docker run -d -p 80:80 --name cast ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker run -d -p 80:81 --name movie ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 10
                    '''
                }
            }
        }
    }
}
