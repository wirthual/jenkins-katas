pipeline {

environment {
        docker_username = 'myusername'
    }

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
            //sh 'ci/build-app.sh'
            //stash includes: '/app/build/libs/', name: 'code'
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

      stage('Master branch build') {
          when { branch "master" }
          steps {
            sh 'Echo "On master branch"'
          }
        }


      stage("Push docker app"){
        environment {
        DOCKERCREDS = credentials('pw') //use the credentials just created in this stage
        }
        steps {
              unstash 'code' //unstash the repository code
              sh 'ci/build-docker.sh'
              sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
              sh 'ci/push-docker.sh'
        }
      }

  }
}