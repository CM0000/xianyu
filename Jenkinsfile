pipeline {
  agent any
  environment {
    ENTERPRISE = "myxianyu"
    PROJECT = "coding-demo"
    ARTIFACT = "coding-demo"
    CODE_DEPOT = "coding-demo"
    
    ARTIFACT_BASE = "${ENTERPRISE}-docker.pkg.coding.net"
    ARTIFACT_IMAGE="${ARTIFACT_BASE}/${PROJECT}/${ARTIFACT}/${CODE_DEPOT}"
  }
  stages {
    stage('检出') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: env.GIT_BUILD_REF]],
                            userRemoteConfigs: [[url: env.GIT_REPO_URL, credentialsId: env.CREDENTIALS_ID]]])
      }
    }
    stage('编译') {
      steps {
        sh './mvnw package -Dmaven.test.skip=true'
      }
    }
    stage('单元测试') {
      steps {
        sh "./mvnw test"
      }
      post {
        always {
        	junit 'target/surefire-reports/*.xml'
        }
      }
    }
    stage('打包镜像') {
      steps {
		sh "docker build -t ${ARTIFACT_IMAGE}:${env.GIT_BUILD_REF} ."
        sh "docker tag ${ARTIFACT_IMAGE}:${env.GIT_BUILD_REF} ${ARTIFACT_IMAGE}:latest"
      }
    }
    stage('推送到制品库') {
      steps {
		script {
          docker.withRegistry("https://${ARTIFACT_BASE}", "${env.DOCKER_REGISTRY_CREDENTIALS_ID}") {
            docker.image("${ARTIFACT_IMAGE}:${env.GIT_BUILD_REF}").push()
       		docker.image("${ARTIFACT_IMAGE}:latest").push()
          }
        }
      }
    }
  }
}