pipeline {

  environment{
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "lastest"
    STAGING = "eazytraining-staging"
    PRODUCTION = "easytraining-production"
  }
  
  agent none
  stages {
      stage('Build') {
          agent any
          steps {
            script {
              sh 'docker build . -t eazytraining/$IMAGE_NAME:$IMAGE_TAG'
            }
          }
      }
      stage('Run') {
          agent any
          steps {
            script {
              sh '''
                docker run -d -p 80:5000 --name=$IMAGE_NAME -e PORT=5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
                sleep 5
              '''
            }
        }
      }
      stage('Test') {
          agent any
          steps {
            script {
              sh '''
                curl http://localhost | grep -q "Hello World"
              '''
            }
        }
      }
    stage('Clean up') {
          agent any
          steps {
            script {
              sh '''
                docker stop $IMAGE_NAME
                docker rm $IMAGE_NAME
              '''
            }
        }
      }
    stage('Push image in staging and deploy it') {
      when {
          expression { GIT_BRANCH == 'origin/master' } 
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        scripts {
          sh '''
            heroku container:login
            heroku create $STAGING || echo "project already exists"
            heroku container:push -a $STAGING web
            heroklu container:release -a $STAGING web
          '''
        }
      }
    }
    stage('Push image in production and deploy it') {
      when {
          expression { GIT_BRANCH == 'origin/master' } 
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        scripts {
          sh '''
            heroku container:login
            heroku create $PRODUCTION || echo "project already exists"
            heroku container:push -a $PRODUCTION web
            heroklu container:release -a $PRODUCTION web
          '''
        }
      }
    }
  }
}
