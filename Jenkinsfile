pipeline{
  agent any
  tools{
    nodejs "nodejs"
  }
  environment{
    SCANNER_HOME=tool "sonar-scanner"
    APP_NAME="netflix"
    DOCKER_USER="rutvikg"
    IMAGE_NAME="${DOCKER_USER}/${APP_NAME}"
  }
  stages{
    stage("cleaning workspace"){
      steps{
        cleanWs()
      }
    }
    stage("cloning workspace"){
      steps{
        git branch: "main", url: "https://github.com/Rutvikgalale/devsecops-netflix-app.git"
      }
    }
    stage("sonarqube scanning"){
      steps{
        withSonarQubeEnv('sonar-server') {
          sh ''' 
          $SCANNER_HOME/bin/sonar-scanner \
          -Dsonar.projectName=netflix \
          -Dsonar.projectKey=netflix 
          '''
        }
      }
    }
    stage("code quality check"){
      steps{
        script{
          waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
        }
      }
    }
    stage("install dependencies"){
      steps{
        sh "npm install"
      }
    }

    stage("owasp dependency check"){
      steps{
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
    stage("trivy FS scan"){
      steps{
        sh "trivy fs . > trivyfs.txt"
      }
    }
    stage("docker build & push"){
      steps{
        script{
          withCredentials([usernamePassword(credentialsId: "dockerhub", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]){
            sh """
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker build --build-arg TMDB_V3_API_KEY=d5ca9ac9c910b2b310aa01e2d794a5e8 -t ${APP_NAME} .
            docker tag ${APP_NAME} ${IMAGE_NAME}
            docker push "${IMAGE_NAME}"
              """
            }
          }
        }
      }
    stage("trivy image scan"){
      steps{
        sh "trivy image ${IMAGE_NAME} > trivyimage.txt"
      }
    }
    stage("deploy application to container"){
      steps{
        sh "docker run -dit --restart unless-stopped --name ${APP_NAME} -p 8081:80 ${IMAGE_NAME}"
      }
    }
    post {
      always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'rutvikgalale@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
      }
    }
  }
}
