
def APP_NAME = "example-build-image"

pipeline {
  agent {
    label('master')
  }

  environment {
    APP_NAME     = "${APP_NAME}"
    SOURCES_ROOT = "."
    // use a real semvar plugin or sth.
    IMAGE_TAG    = "${env.BUILD_NUMBER}.0.0"
    EXTERNAL_DOCKER_REGISTRY = "cs-prod-tools-artifactory-01.ngis.zone:5012"
  }

  stages {
    stage('build and publish docker image to openshift') {
      steps {
        echo 'Building Docker image.'

        sh '''
          oc get bc/${APP_NAME} \
            || oc new-build \
                --binary=true  \
                --name="${APP_NAME}" \
                --strategy="docker"
        '''

        sh '''
          oc start-build ${APP_NAME} \
            --from-dir=${SOURCES_ROOT} \
            --follow=true \
            --wait=true
        '''

        sh '''
          oc tag ${APP_NAME}:latest ${APP_NAME}:${IMAGE_TAG} 
        '''
      }
    }

    stage('deploy docker image to openshift') {
      steps {
        sh '''
          oc get dc/${APP_NAME} || oc new-app \
            --name=${APP_NAME} \
            --image-stream="${APP_NAME}:${IMAGE_TAG}" \
            -e EVN="VAR" 
        '''
      }
    }

    stage('build and publish docker image to external registry') {
      steps {
        echo 'Building Docker image.'

        sh '''
          oc get bc/ext-reg-${APP_NAME} \
            || oc new-build \
                --binary=true  \
                --name="ext-reg-${APP_NAME}" \
                --strategy="docker" \
                --to="${EXTERNAL_DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}" 
        '''

        sh '''
          oc start-build ext-reg-${APP_NAME} \
            --from-dir=${SOURCES_ROOT} \
            --follow=true \
            --wait=true
        '''

        sh '''
          oc tag ${APP_NAME}:latest ${APP_NAME}:${IMAGE_TAG} 
        '''
      }
    }
  }
}