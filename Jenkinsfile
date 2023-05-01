pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
       stage('Maven test') {
            steps {
              sh "mvn test"
              }
              post{
                always{
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec'
                }
              }
        }
        stage('docker build') {
          steps {
            withDockerRegistry([credentialsId: "dockerhub", url: ""]){
              sh "printenv"
              sh "docker build -t ramki0610/numeric-app:""$GIT_COMMIT"" ."
              sh "docker push ramki0610/numeric-app:""$GIT_COMMIT""
            }

          }
        }  
    }
}

