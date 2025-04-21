pipeline {
    environment {  
        DOCKER_ID = "cmoisby"  
        DOCKER_IMAGE_CAST = "cast"
        DOCKER_IMAGE_MOVIE = "movie"
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
                        docker rm -f cast
                        docker rm -f movie
                        echo "${DOCKER_ID}/${DOCKER_IMAGE_CAST}"
                        docker build -t ${DOCKER_ID}/${DOCKER_IMAGE_CAST}:${DOCKER_TAG} ./cast-service
                        echo "${DOCKER_ID}/${DOCKER_IMAGE_MOVIE}"
                        docker build -t ${DOCKER_ID}/${DOCKER_IMAGE_MOVIE}:${DOCKER_TAG} ./movie-service
                        sleep 6
                    '''
                }
            }
        }

        stage('Docker run') { // Run containers from our built images
            steps {
                script {
                    sh '''
                        docker run -d -p 8001:80 --name cast ${DOCKER_ID}/${DOCKER_IMAGE_CAST}:${DOCKER_TAG}
                        docker run -d -p 8002:80 --name movie ${DOCKER_ID}/${DOCKER_IMAGE_MOVIE}:${DOCKER_TAG}
                        sleep 10
                    '''
                }
            }
        }
         stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    echo 'Testing CAST'
                    curl localhost:8001
                     echo 'Testing MOVIE'
                    curl localhost:8001
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                '''
                }
            }

        }
 
    }
}
