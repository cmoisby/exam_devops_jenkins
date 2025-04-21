pipeline {
environment {  
DOCKER_ID = "cmoisby"  
DOCKER_IMAGE = "Jenkins_devops_exams"
DOCKER_TAG = "v.${BUILD_ID}.0"  
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f cast-service
                 docker rm -f movie-service
                 echo '$DOCKER_ID/$DOCKER_IMAGE'
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG ./cast
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG ./movie
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 80:80 --name cast $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    ocker run -d -p 80:81 --name movie $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }
         }
}