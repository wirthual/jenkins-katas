pipeline {
  agent any
  stages {
    stage("clone down"){
        agent {
          label 'swarm'
        }
        steps{
            stash excludes: '.git', name: 'code'
        }
    }

    stage('Parallel Execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "Hello World"'
          }
        }

        stage('Build App') {
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          options{skipDefaultCheckout(true)}

          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts '/app/build/libs/'
          }
        }

        stage('Test App') {
                  agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          options{skipDefaultCheckout(true)}
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }

  }
}
