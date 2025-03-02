pipeline {
    agent any 
stages {
    stage ('build stage') {
        steps {
            sh 'mvn clean install'
        }
    }
    stage ('build docker image and tag the image') {
        steps {
            sh '''
            docker build -t skb-bank-app:v.${BUILD_NUMBER} .
            docker tag skb-bank-app:v.${BUILD_NUMBER} khadar3099/skb-bank-app:v.${BUILD_NUMBER}
            '''
        }
    }
    stage ('push docker image to docker hub') {
        steps {
            withCredentials([string(credentialsId: 'dockerhubpswd', variable: 'dockerpswd')]) {
                sh 'docker login -u khadar3099 -p ${dockerpswd}'
                sh 'docker push khadar3099/skb-bank-app:v.${BUILD_NUMBER}'
                sh 'docker rmi skb-bank-app:v.${BUILD_NUMBER}'
                sh 'docker rmi khadar3099/skb-bank-app:v.${BUILD_NUMBER}'
            }

        }
    }
    stage ('deploy docker image or run container in ec2 instance') {
        steps {
            sh 'docker ps -q -f name=SKB-Bank_container && docker stop SKB-Bank_container && docker rm SKB-Bank_container || echo "Container not found or already stopped."'
            sh 'docker run -d -p 8181:8181 --name SKB-Bank_container khadar3099/skb-bank-app:v.${BUILD_NUMBER}'
          }
      }
  }   
}
