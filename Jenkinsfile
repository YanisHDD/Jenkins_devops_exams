pipeline {
    environment {
        DOCKER_ID = "yanishdd"  // Ton Docker ID
        DOCKER_IMAGE_1 = "cast_service"  // Première image Docker
        DOCKER_IMAGE_2 = "movie_service"  // Deuxième image Docker
        DOCKER_TAG = "v.${BUILD_ID}.0"  // Tag de l'image basé sur l'ID du build
    }

    agent any  // Jenkins peut sélectionner n'importe quel agent disponible

    stages {
        stage('Docker Build') {  // Construire les images Docker
            steps {
                script {
                    // Construire les images Docker à partir des Dockerfiles
                    sh '''
                        docker rm -f jenkins || true
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_1:$DOCKER_TAG -f ./cast-service/Dockerfile .
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_2:$DOCKER_TAG -f ;/movie-service/Dockerfile .
                        sleep 6
                    '''
                }
            }
        }

        stage('Docker Push') {  // Push des images Docker sur DockerHub
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")  // Récupérer le mot de passe DockerHub depuis Jenkins
            }

            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE_1:$DOCKER_TAG
                        docker push $DOCKER_ID/$DOCKER_IMAGE_2:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deployment to Dev') {  // Déploiement sur l'environnement Dev
            environment {
                KUBECONFIG = credentials("config")  // Récupérer le kubeconfig depuis Jenkins
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        helm upgrade --install app fastapi --set service.nodePort=30008 --namespace dev --set image.tag=${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deployment to Staging') {  // Déploiement sur l'environnement Staging
            environment {
                KUBECONFIG = credentials("config")  // Récupérer le kubeconfig depuis Jenkins
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        helm upgrade --install app fastapi --set service.nodePort=30009 --namespace staging --set image.tag=${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deployment to Prod') {  // Déploiement sur l'environnement Prod
            environment {
                KUBECONFIG = credentials("config")  // Récupérer le kubeconfig depuis Jenkins
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {  // Validation manuelle avant déploiement en prod
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        helm upgrade --install app fastapi --set service.nodePort=30011 --namespace prod --set image.tag=${DOCKER_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }

        success {
            echo 'Pipeline succeeded!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
