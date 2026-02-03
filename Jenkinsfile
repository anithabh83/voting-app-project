pipeline {
  agent any

  environment {
    DOCKERHUB_USER = "YOUR_DOCKERHUB_USERNAME"
    TAG = ""
    NAMESPACE = ""
  }

  stages {

    /* ---------------------------
       Detect Environment
       --------------------------- */
    stage('Set Environment') {
      steps {
        script {
          if (env.BRANCH_NAME == 'dev') {
            TAG = 'dev'
            NAMESPACE = 'dev'
          } else if (env.BRANCH_NAME == 'test') {
            TAG = 'test'
            NAMESPACE = 'test'
          } else if (env.BRANCH_NAME == 'prod') {
            TAG = 'prod'
            NAMESPACE = 'prod'
          } else {
            error "Branch not supported"
          }
        }
      }
    }

    /* ---------------------------
       Build Docker Images
       --------------------------- */
    stage('Build Docker Images') {
      steps {
        sh """
        docker build -t $DOCKERHUB_USER/vote:$TAG vote/
        docker build -t $DOCKERHUB_USER/worker:$TAG worker/
        docker build -t $DOCKERHUB_USER/result:$TAG result/
        """
      }
    }

    /* ---------------------------
       Push Images to DockerHub
       --------------------------- */
    stage('Push Docker Images') {
      steps {
        sh """
        docker push $DOCKERHUB_USER/vote:$TAG
        docker push $DOCKERHUB_USER/worker:$TAG
        docker push $DOCKERHUB_USER/result:$TAG
        """
      }
    }

    /* ---------------------------
       Create Namespace Automatically
       --------------------------- */
    stage('Create Namespace') {
      steps {
        sh """
        kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE
        """
      }
    }

    /* ---------------------------
       Apply ResourceQuota
       --------------------------- */
    stage('Apply Resource Quota') {
      steps {
        sh """
        kubectl apply -f k8s/resource-quota.yaml -n $NAMESPACE
        """
      }
    }

    /* ---------------------------
       Deploy to Kubernetes
       --------------------------- */
    stage('Deploy Application') {
      steps {
        sh """
        kubectl apply -f k8s/deployment.yaml -n $NAMESPACE
        kubectl apply -f k8s/service.yaml -n $NAMESPACE
        """
      }
    }
  }
}

