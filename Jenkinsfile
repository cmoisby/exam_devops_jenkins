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
