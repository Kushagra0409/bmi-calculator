def app

pipeline {
    agent any

    environment {
      DOCKER_IMAGE_NAME = "raylayadi/bmi-calculator"
      DOCKER_REGISTRY = "https://registry.hub.docker.com"
      DOCKER_LOGIN_JENKINS_CREDENTIALS = "docker-login"
      K8S_KUBECONFIG_JENKINS_CREDENTIALS = "k8s-bmi-calculator"
      K8S_BMI_CALCULATOR_DEPLOY_NAME = "bmi-calculator"
      K8S_BMI_CALCULATOR_CONTAINER_NAME = "bmi-calculator"
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './scripts/bmi.test.sh'
            }
        }
        stage('Deploy') {
            steps {
                script {
                   // Only build the docker image if the test is successful
                   app = docker.build("${env.DOCKER_IMAGE_NAME}")

                   // Push docker image to the docker registry
                   docker.withRegistry("${env.DOCKER_REGISTRY}", "${env.DOCKER_LOGIN_JENKINS_CREDENTIALS}") {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                   }

                   // Update the deployment image which will automatically perform rolling update
                   withKubeConfig([credentialsId: "${env.K8S_KUBECONFIG_JENKINS_CREDENTIALS}"]) {
                      sh "kubectl set image deployment/${env.K8S_BMI_CALCULATOR_DEPLOY_NAME} ${env.K8S_BMI_CALCULATOR_CONTAINER_NAME}=${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                   }
                }
            }
        }
    }
}