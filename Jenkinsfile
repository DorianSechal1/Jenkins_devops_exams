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
                    docker build -t $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG -t $DOCKER_ID/$MOVIE_SERVICE_IMAGE:latest ./movie-service
                    docker build -t $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG -t $DOCKER_ID/$CAST_SERVICE_IMAGE:latest ./cast-service
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
                    docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:latest
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:latest
                    '''
                }
            }
        }
        stage('Deploy to Dev') {
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
                      cp helm/values.yaml values.yml
                      cat values.yml
                      cat values.yml
                      helm upgrade --install dev-app ./helm --values=values.yml --namespace dev
                      '''
                      }
                  }
          }
        stage('Deploy to QA') {
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
                      cp helm/values.yaml values.yml
                      cat values.yml
                      helm upgrade --install qa-app ./helm --values=values.yml --namespace qa
                      '''
                      }
                  }
          }
        stage('Deploy to Staging') {
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
                      cp helm/values.yaml values.yml
                      cat values.yml
                      helm upgrade --install staging-app ./helm --values=values.yml --namespace staging
                      '''
                      }
                  }
          }
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
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
                      cp helm/values.yaml values.yml
                      cat values.yml
                      helm upgrade --install prod-app ./helm --values=values.yml --namespace prod
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
