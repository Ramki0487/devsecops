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

        // stage('sonarqube-SAST'){
        //   steps {
        //           withSonarQubeEnv('sonarserver') {
        //             sh "mvn sonar:sonar -Dsonar.projectKey=numeric-app -Dsonar.host.url=http://devsecopsk8s.eastus.cloudapp.azure.com:9000/"
        //           }
        //           timeout(time: 2, unit: 'MINUTES') {
        //             script {
        //               waitForQualityGate abortPipeline: true
        //             }
        //           }
        //   }     
        // }

        stage('dependency check'){
          steps {
            sh "mvn dependency-check:check"
          }
          post {
            always {
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
          }
        }
        stage('docker build') {
          steps {
            withDockerRegistry([credentialsId: "dockerhub", url: ""]){
              sh 'printenv'
              sh 'docker build -t ramki0610/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push ramki0610/numeric-app:""$GIT_COMMIT""'
            }

          }
        }
          stage('kubernetes deployment'){
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']){
                sh "sed -i 's#replace#ramki0610/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh 'cat k8s_deployment_service.yaml'
                sh "kubectl apply -f k8s_deployment_service.yaml"
                sh "kubectl get po"
              }
            }
        } 
    }
}

