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

        stage(sonarqube-SAST){
          steps {
                  sh "mvn clean verify sonar:sonar \
                  -Dsonar.projectKey=numeric-app \
                  -Dsonar.projectName='numeric-app' \
                  -Dsonar.host.url=http://devsecopsk8s.eastus.cloudapp.azure.com:9000 \
                  -Dsonar.token=sqp_04012b21d9370b294a9da173c99f741866d982a1"
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

