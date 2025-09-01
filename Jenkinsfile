pipeline {
  agent any
  environment { LOCAL_IMAGE = "flask-hello:${BUILD_NUMBER}" }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') { steps { sh 'docker build -t $LOCAL_IMAGE .' } }
    stage('Unit tests') { steps { sh 'docker run --rm $LOCAL_IMAGE pytest tests -q' } }
    stage('Tag & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
          sh 'docker tag "$LOCAL_IMAGE" "$DOCKER_USER/flask-hello:$BUILD_NUMBER"'
          sh 'docker tag "$LOCAL_IMAGE" "$DOCKER_USER/flask-hello:latest"'
          sh 'docker push "$DOCKER_USER/flask-hello:$BUILD_NUMBER"'
          sh 'docker push "$DOCKER_USER/flask-hello:latest"'
        }
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'printf "IMAGE=%s\\n" "$DOCKER_USER/flask-hello:latest" > deploy/.env'
          sh 'docker compose -f deploy/docker-compose.yml pull'
          sh 'docker compose -f deploy/docker-compose.yml up -d'
        }
      }
    }
  }
}
