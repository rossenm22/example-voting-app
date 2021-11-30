pipeline {
  agent none
  stages {
    stage('worker build') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Compiling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker test') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Unit Tests on Worker app'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'Packaging Worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker docker-package') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'Packaging Worker app with Docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def workerImage = docker.build("rossenm222/worker:v${env.BUILD_ID}","./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('result build') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compiling result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result test') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on Result app'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('result docker-package') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Packaging Result app with Docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def workerImage = docker.build("rossenm222/result:v${env.BUILD_ID}","./result")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('vote build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compiling vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on Vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote docker-package') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'Packaging Vote app with Docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def workerImage = docker.build("rossenm222/vote:v${env.BUILD_ID}","./vote")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('Deploy to Dev') {
      agent any
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Build pipeline for instavote app is complete..'
    }

    failure {
      slackSend(message: "Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

    success {
      slackSend(message: "Build successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

  }
}