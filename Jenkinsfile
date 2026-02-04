pipeline {
  // Run this pipeline on any available Jenkins agent
  agent any

  environment {
    // DockerHub username used for tagging and pushing images
    DOCKERHUB_USER = "abhanumantharaju"
  }

  stages {

    /* ---------------------------
       Detect Environment based on Git branch
       --------------------------- */
    stage('Set Environment') {
      steps {
        script {
          // If pipeline is triggered from dev branch
          // set Docker image tag and Kubernetes namespace as dev
          if (env.BRANCH_NAME == 'dev') {
            env.TAG = 'dev'
            env.NAMESPACE = 'dev'

          // If triggered from test branch
          // set tag and namespace as test
          } else if (env.BRANCH_NAME == 'test') {
            env.TAG = 'test'
            env.NAMESPACE = 'test'

          // If triggered from prod branch
          // set tag and namespace as prod
          } else if (env.BRANCH_NAME == 'prod') {
            env.TAG = 'prod'
            env.NAMESPACE = 'prod'

          // Fail the pipeline if branch is not supported
          } else {
            error "Branch not supported"
          }
        }
      }
    }

    /* ---------------------------
       Docker Login using Jenkins credentials
       --------------------------- */
    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          // Login to DockerHub securely without exposing password
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
        // Build vote service image with branch-based tag
        docker build -t $DOCKERHUB_USER/vote:$TAG vote/

        // Build worker service image with same tag
        docker build -t $DOCKERHUB_USER/worker:$TAG worker/

        // Build result service image with same tag
        docker build -t $DOCKERHUB_USER/result:$TAG result/
        """
      }
    }

    /* ---------------------------
       Push Docker Images to DockerHub
       --------------------------- */
    stage('Push Docker Images') {
      steps {
        sh """
        // Push all built images to DockerHub
        docker push $DOCKERHUB_USER/vote:$TAG
        docker push $DOCKERHUB_USER/worker:$TAG
        docker push $DOCKERHUB_USER/result:$TAG
        """
      }
    }

    /* ---------------------------
       Create Kubernetes Namespace if not exists
       --------------------------- */
    stage('Create Namespace') {
      steps {
        sh """
        // Check if namespace exists
        // If not, create it automatically
        kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE
        """
      }
    }

    /* ---------------------------
       Apply ResourceQuota to namespace
       --------------------------- */
    stage('Apply Resource Quota') {
      steps {
        sh """
        // Apply resource quota to control CPU and memory usage
        kubectl apply -f k8s/resource-quota.yaml -n $NAMESPACE
        """
      }
    }

    /* ---------------------------
       Deploy application to Kubernetes
       --------------------------- */
    stage('Deploy Application') {
      steps {
        sh """
        // Deploy application pods
        kubectl apply -f k8s/deployment.yaml -n $NAMESPACE

        // Expose services inside the cluster
        kubectl apply -f k8s/service.yaml -n $NAMESPACE
        """
      }
    }
  }
}
