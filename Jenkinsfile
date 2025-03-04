pipeline {
  agent none

  stages {
    stage('Checkstyle') {
      when {
        changeRequest()
      } 
      agent {
        docker { image 'maven:3.8.5-openjdk-17' }
       }
      steps {
        checkout scm
        sh 'mvn checkstyle:checkstyle'
      }
      post {
        always {
          archiveArtifacts artifacts: 'target/checkstyle-result.xml', allowEmptyArchive: true
        }
      }
    }
    stage('Test') {
      when {
        changeRequest()
      } 
 
      agent {
        docker { image 'maven:3.8.5-openjdk-17' }
       }
      steps {
        sh 'mvn test -Dcheckstyle.skip=true'
      }
    }
    stage('Build') {
      when {
        changeRequest()
      } 
 
      agent {
        docker { image 'maven:3.8.5-openjdk-17' }
      }
      steps {
        sh 'mvn clean package -DskipTests -Dcheckstyle.skip=true'
      }
      post {
        always {
          archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
        }
      }
    }
    stage('Build Image') {
      when {
        changeRequest()
      } 
 
      agent {
        docker {
          image 'docker:20.10.16'
          args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
        }
      }
      steps {
        script {
          def docker_image=docker.build("mr:8084/spring-petclinic:$GIT_COMMIT")
          sh 'docker login -u "$REGISTRY_USER" -p "$REGISTRY_PASS" mr:8084'
          docker.withRegistry('http://mr:8084') {
            docker_image.push()
          }
        }
      }
    }
    stage('Build Image Main') {
      when {
        not { changeRequest() }
      } 
 
      agent {
        docker {
          image 'docker:20.10.16'
          args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
        }
      }
      steps {
        script {
          def docker_image=docker.build("main:8083/spring-petclinic:$GIT_COMMIT")
          sh 'docker login -u "$REGISTRY_USER" -p "$REGISTRY_PASS" main:8083'
          docker.withRegistry('http://main:8083') {
            docker_image.push()
          }
        }
      }
    }
  }
}
