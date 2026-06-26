pipeline {
  agent {
    kubernetes {
      label 'kaniko'
      defaultContainer 'kaniko'
    }
  }

  environment {
    REPO  = 'k8s-test-core-server'
    IMAGE = "ghcr.io/${env.GH_ORG.toLowerCase()}/${REPO}"
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Build & Push') {
      steps {
        sh '''
          /kaniko/executor \
            --context=dir://${WORKSPACE} \
            --dockerfile=Dockerfile \
            --destination=${IMAGE}:${GIT_COMMIT} \
            --destination=${IMAGE}:latest \
            --build-arg=GIT_SHA=${GIT_COMMIT} \
            --cache=true \
            --cache-repo=${IMAGE}/cache \
            --cache-ttl=168h \
            --customPlatform=linux/arm64 \
            --snapshot-mode=redo \
            --use-new-run
        '''
      }
    }
  }
}
