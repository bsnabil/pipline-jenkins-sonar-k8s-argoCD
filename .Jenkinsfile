pipeline {
  agent {
    docker {
      image 'nabil2222/react-application' //unable to pull
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        git branch: 'main', url: 'https://github.com/bsnabil/pipline-jenkins-sonar-k8s-argoCD.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd spring-boot-app/ && mvn clean package'
      }
    }
 
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "nabil2222/spring-boot-app-service"
        // DOCKERFILE_LOCATION = "spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'cd spring-boot-app/ && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
             GIT_REPO_NAME = "pipline-jenkins-sonar-k8s-argoCD/"
             GIT_USER_NAME = "bsnabil"
        }
         steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "nabil.bensalem22222@gmail.com"
                    git config user.name "bsnabil"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml 
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
