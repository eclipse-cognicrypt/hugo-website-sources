pipeline {

  agent {
    kubernetes {
      label 'hugo-agent'
      yaml """
apiVersion: v1
metadata:
  labels:
    run: hugo
  name: hugo-pod
spec:
  containers:
    - name: hugo
      image: eclipsecbi/hugo:0.42.1
      command:
      - cat
      tty: true
"""
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    checkoutToSubdirectory('hugo')
  }

  triggers {
    cron('H/5 * * * *')
  }

  stages {
    stage('Checkout www repo') {
      steps {
        dir('www') {
            git branch: 'master',
                url: 'ssh://genie.cognicrypt@git.eclipse.org:29418/www.eclipse.org/cognicrypt.git',
                credentialsId: 'git.eclipse.org-bot-ssh'
        }
      }
    }
    stage('Build website with Hugo') {
      steps {
        container('hugo') {
            dir('hugo') {
                sh 'hugo -b https://www.eclipse.org/cognicrypt/'
            }
        }
      }
    }
    stage('Push to master branch') {
      steps {
        sh 'cp -Rvf hugo/public/* www/'
        dir('www') {
            sshagent(['git.eclipse.org-bot-ssh']) {
                sh '''
                git add -A
                if ! git diff --cached --exit-code; then
                  echo "Changes have been detected, publishing to repo 'www.eclipse.org/cognicrypt'"
                  git config --global user.email "cognicrypt-bot@eclipse.org"
                  git config --global user.name "Cognicrypt Bot"
                  git commit -m "Website build ${JOB_NAME}-${BUILD_NUMBER}"
                  git log --graph --abbrev-commit --date=relative -n 5
                  git push origin HEAD:master
                else
                  echo "No change have been detected since last build, nothing to publish"
                fi
                '''
            }
        }
      }
    }
  }
}