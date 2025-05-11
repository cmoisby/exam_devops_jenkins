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
        stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install  app  ./charts --values=values.yml --namespace dev
                '''
                }
            }

        } 
        stage('Test Acceptance DEV') {
         steps {
             script {
            // Récupérer dynamiquement le nodePort
            def nodePort = sh(script: "kubectl get svc nginx -n dev -o jsonpath='{.spec.ports[0].nodePort}'", returnStdout: true).trim()

            // Vérifier la réponse du service avec curl 
            def response = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" http://localhost:$nodePort/api/v1/movies/docs", returnStdout: true).trim()
            if (response != '200') {
                error "App non prête ! Code HTTP: ${response}"
            } else {
                echo "App OK l'application est disponible sur le port ${nodePort}"
                 
            }
              }
             }
        }
          stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install  app  ./charts --values=values.yml --namespace qa
                '''
                }
            }

        } 
        stage('Test Acceptance QA') {
         steps {
             script {
            // Récupérer dynamiquement le nodePort
            def nodePort = sh(script: "kubectl get svc nginx -n qa -o jsonpath='{.spec.ports[0].nodePort}'", returnStdout: true).trim()

            // Vérifier la réponse du service avec curl 
            def response = sh(script: "curl -s -o /qa/null -w \"%{http_code}\" http://localhost:$nodePort/api/v1/movies/docs", returnStdout: true).trim()
            if (response != '200') {
                error "App non prête ! Code HTTP: ${response}"
            } else {
                echo "App OK l'application est disponible sur le port ${nodePort}"
                 
            }
              }
             }
        }
        stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ./charts --values=values.yml --namespace staging
                '''
                }
            }

        }
         stage('Test Acceptance staging') {
         steps {
             script {
            // Récupérer dynamiquement le nodePort
            def nodePort = sh(script: "kubectl get svc nginx -n staging -o jsonpath='{.spec.ports[0].nodePort}'", returnStdout: true).trim()

            // Vérifier la réponse du service avec curl 
            def response = sh(script: "curl -s -o /staging/null -w \"%{http_code}\" http://localhost:$nodePort/api/v1/movies/docs", returnStdout: true).trim()
            if (response != '200') {
                error "App non prête ! Code HTTP: ${response}"
            } else {
                echo "App OK l'application est disponible sur le port ${nodePort}"
                 
            }
              }
             }
        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ./charts --values=values.yml --namespace prod
                '''
                }
            }

        }
    }

    post {
    success {
        echo "This will run if the job succeeded"
        mail to: "souheilby@gmail.com",
            subject: "${env.JOB_NAME} - Build #${env.BUILD_ID} succeeded",
            body: "Good news! The build for ${env.JOB_NAME} completed successfully. Check details at ${env.BUILD_URL}"
        }
    
        failure {
            echo "This will run if the job failed"
            mail to: "souheilby@gmail.com",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }

}
 
  