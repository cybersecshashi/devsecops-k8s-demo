pipeline {
  agent any

  stages {
      stage('Build Artifact - Maven') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }

      stage('Unit Tests - Junit & Jacoco') {
            steps {
              sh "mvn test"
            }
            post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
        }  

      stage('SonarQube - SAST') {
        steps {
          withSonarQubeEnv('SonarQube') {
            sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devsecops-shashi.eastus.cloudapp.azure.com:9000"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
        }
      }  

    //   stage('Vulnerability Scan - Docker ') {
    //   steps {
    //     sh "mvn dependency-check:check"
    //   }
    //   post {
    //     always {
    //       dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
    //     }
    //   }
    // } 

      stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t sharminator/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push sharminator/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#sharminator/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }

    }
}
