pipeline {
    agent any
    tools {
        maven "maven-3.9"
    }
    stages {
        stage("build jar") {
            steps {
                script {
                    echo "building the application..."
                    sh "mvn package"
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: "docker-hub-repo", passwordVariable: "PASS", usernameVariable: "USER")]) {
                        sh "docker build -t prince450/demo-app:jma-2.0 ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push prince450/demo-app:jma-2.0"
                    }
                 }
            }
        }
        stages {
        stage("deploy") {
            steps {
                script {
                    echo ""
                }
            }
          }
      
        }                 
  }      
}                    