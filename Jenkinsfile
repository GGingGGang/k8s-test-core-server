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
    stage('Skip check') {
      steps {
        container('jnlp') {
          script {
            def msg = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
            env.CI_SKIP = msg.contains('[ci skip]') ? 'true' : 'false'
            if (env.CI_SKIP == 'true') {
              echo 'Bump commit ([ci skip]) — build/bump 스킵, 루프 종료.'
            }
          }
        }
      }
    }

    stage('Build & Push') {
      when { expression { env.CI_SKIP == 'false' } }
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
            --use-new-run \
            --ignore-path=/busybox \
            --ignore-path=/home/jenkins
        '''
      }
    }

    stage('Bump manifest') {
      when { expression { env.CI_SKIP == 'false' } }
      steps {
        container('jnlp') {
          withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
            sh '''
              set +x
              sed -i "s|newTag:.*|newTag: ${GIT_COMMIT}|" deploy/k8s/kustomization.yaml
              git config user.email "ci@ggang.cloud"
              git config user.name "jenkins-ci"
              git add deploy/k8s/kustomization.yaml
              git commit -m "ci: bump core to ${GIT_COMMIT} [ci skip]"
              git push "https://${GIT_USER}:${GIT_TOKEN}@github.com/${GH_ORG}/${REPO}.git" HEAD:main
            '''
          }
        }
      }
    }
  }
}
