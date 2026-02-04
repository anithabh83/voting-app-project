pipeline {
  agent any

  environment {
    // DockerHub username
    DOCKERHUB_USER = "abhanumantharaju"
  }

  stages {

    /* ---------------------------
       Set environment based on branch
       --------------------------- */
    stage('Set Environment') {
      steps {
        script {
          if (env.BRANCH_NAME == 'dev') {
            env.TAG = 'dev'
            env.NAMESPACE = 'dev'
          } else if (env.BRANCH_NAME == 'test') {
            env.TAG = 'test'
            env.NAMESPACE = 'test'
          } else if (env.BRANCH_NAME == 'prod') {
            env.TAG = 'prod'
            env.NAMESPACE = 'prod'
          } else {
            error "Unsupported branch"
          }
        }
      }
    }

    /* ---------------------------
       Docker Login
       --------------------------- */
    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
          '''
        }
      }
    }

    /* ---------------------------
       Build Docker Images
       --------------------------- */
    stage('Build Docker Images') {
      steps {
        sh """
          # Build Docker images with branch-based tag
          docker build -t $DOCKERHUB_USER/vote:$TAG vote/
          docker build -t $DOCKERHUB_USER/worker:$TAG worker/
          docker build -t $DOCKERHUB_USER/result:$TAG result/
        """
      }
    }

    /* ---------------------------
       Push Docker Images
       --------------------------- */
    stage('Push Docker Images') {
      steps {
        sh """
          # Push images to DockerHub
          docker push $DOCKERHUB_USER/vote:$TAG
          docker push $DOCKERHUB_USER/worker:$TAG
          docker push $DOCKERHUB_USER/result:$TAG
        """
      }
    }

    /* ---------------------------
       Create Kubernetes Namespace
       --------------------------- */
    stage('Create Namespace') {
      steps {
        sh """
          # Create namespace if it does not exist
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
          # Apply resource quota to control resource usage
          kubectl apply -f k8s/resource-quota.yaml -n $NAMESPACE
        """
      }
    }

    /* ---------------------------
       Deploy Application
       --------------------------- */
    stage('Deploy Application') {
      steps {
        sh """
          # Deploy application components
          kubectl apply -f k8s/deployment.yaml -n $NAMESPACE
          kubectl apply -f k8s/service.yaml -n $NAMESPACE
        """
      }
    }
  }
}
