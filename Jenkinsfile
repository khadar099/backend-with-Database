pipeline { 
    agent {
        label 'slave'
    }

    tools {
        maven 'maven3'
    }

    environment {
        SONAR_HOME= tool 'sonar-scanner'
    }  

    stages {
        
        stage('Git checkout') {
            steps {
              git branch: 'main', url: 'https://github.com/haleem00/Multi-Tier-With-Database.git'
            }
        }
       

        stage('Compile') {
            steps {
            sh  "mvn compile"
            }
        }
       

        stage('Test') {
            steps {
            sh  "mvn test -DskipTests=true"
            }
        }
       

        stage('Trivy files scan') {
            steps {
              sh  "trivy fs --format table  -o fs.html ."
            }
        }
       

        stage('Sonarqube') {
            steps {
             withSonarQubeEnv('sonar') {
                sh '$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=multi-tier -Dsonar.projectKey=multi-tier -Dsonar.java.binaries=target'
              }
            }
        }


       stage('Build') {
            steps {
             sh "mvn package -DskipTests=true"
            }
        }
       

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh  "mvn deploy -DskipTests=true"
               }
            }
        }
       

        stage('Build docker image') {
            steps {
              script {
                withDockerRegistry(credentialsId: 'hub-token', toolName: 'docker') {
                  sh "docker build -t haleemo/multitier:latest ."
                   }
                }     
            }
        }

        

        stage('Trivy image scan') {
            steps {
            sh  "trivy image --format table -o image.html haleemo/multitier:latest"
            }
        }


        stage('push docker image') {
            steps {
              script {
                withDockerRegistry(credentialsId: 'hub-token', toolName: 'docker') {
                  sh "docker push  haleemo/multitier:latest"
                  }
                }
            }
        }


        stage('Deploy to k8') {
            steps {
              withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://831031BD08A3ABE394AC51296790C352.gr7.us-east-1.eks.amazonaws.com']]) {
                sh "kubectl apply -f ds.yml"
               }
            }
        }
       
        }    }
