pipeline {
  agent any
  environment {
    TAG = "${env.GIT_COMMIT.take(7)}"
    REGISTRY = "registry"
  }
  stages {
    stage('Build') {
      steps {
        sh 'docker build -t gateway:$TAG services/gateway'
      }
    }
    stage('Push') {
      steps {
        sh './scripts/push_images.sh $TAG'
      }
    }
    stage('Deploy') {
      steps {
        sh 'kubectl set image deploy/gateway gateway=$REGISTRY/gateway:$TAG -n idea-validator'
      }
    }
    stage('Smoke') {
      steps {
        sh './scripts/smoke_test.sh'
      }
    }
    stage('Eval') {
      when { triggeredBy 'TimerTrigger' }
      steps {
        sh 'kubectl apply -f k8s/eval-job.yaml -n idea-validator && ./scripts/check_eval.sh'
      }
    }
  }
  post {
    failure {
      echo 'Pipeline failed — nothing promoted'
    }
  }
}
