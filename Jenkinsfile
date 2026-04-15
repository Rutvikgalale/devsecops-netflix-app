pipeline{
  agent any
  tools{
    nodejs "nodejs"
  }
  environment{
    SCANNER_HOME=tool "sonar-scanner"
    app_name="netflix"
    DOCKER_USER="rutvikg"
    image_name="${DOCKER_USER}/${app_name}"
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

 //  stage("owasp dependency check"){
 //     steps{
 //       dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp'
 //       dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
 //     }
 //   }
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
            docker build --build-arg TMDB_V3_API_KEY=d5ca9ac9c910b2b310aa01e2d794a5e8 -t ${app_name} .
            docker tag ${app_name} ${image_name}
            docker push "${image_name}"
              """
            }
          }
        }
      }
    stage("trivy image scan"){
      steps{
        sh "trivy image rutvikg/netflix:latest > trivyimage.txt"
      }
    }
    stage("deploy application to container"){
      steps{
        sh "docker run -dit --name netflix -p 8081:80 rutvikg/netflix:latest"
      }
    }
  }
}
