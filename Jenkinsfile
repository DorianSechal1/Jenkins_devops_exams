pipeline {
    environment {
        DOCKER_ID = "doriansechal"
        MOVIE_SERVICE_IMAGE = "movie-service"
        CAST_SERVICE_IMAGE = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG ./movie-service
                    docker build -t $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG ./cast-service
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
            }
            steps {
                script {
                    sh '''
                    docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Dev') {
            steps {
                script {
                    sh '''
                    rm -rf .kube
                    mkdir .kube
                    echo ${config} > .kube/config
                    cp ./helm/dev/values.yaml values.yml
                    sed -i "s+image.tag.*+image.tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install dev-app ./helm/charts/app --values values.yml --namespace dev
                    '''
                }
            }
        }
        stage('Deploy to QA') {
            steps {
                script {
                    sh '''
                    rm -rf .kube
                    mkdir .kube
                    echo ${config} > .kube/config
                    cp ./helm/qa/values.yaml values.yml
                    sed -i "s+image.tag.*+image.tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install qa-app ./helm/charts/app --values values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                script {
                    sh '''
                    rm -rf .kube
                    mkdir .kube
                    echo ${config} > .kube/config
                    cp ./helm/staging/values.yaml values.yml
                    sed -i "s+image.tag.*+image.tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install staging-app ./helm/charts/app --values values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Voulez-vous déployer en production ?', ok: 'Déployer'
                }
                script {
                    sh '''
                    rm -rf .kube
                    mkdir .kube
                    echo ${config} > .kube/config
                    cp ./helm/prod/values.yaml values.yml
                    sed -i "s+image.tag.*+image.tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install prod-app ./helm/charts/app --values values.yml --namespace prod
                    '''
                }
            }
        }
    }
    post {
        failure {
            mail to: "dorian.sechal@orange.com",
                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                 body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
