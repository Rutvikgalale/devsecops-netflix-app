pipeline{
  agent any
  tools{
    nodejs "nodejs"
  }
  environment{
    SCANNER_HOME=tool "sonar-scanner"
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
/*
    stage("owasp dependency check"){
      steps{
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
*/
    stage("trivy FS scan"){
      steps{
        sh "trivy fs . > trivyfs.txt"
      }
    }
    stage("docker build & push"){
      steps{
        script{
          withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
            sh """
              echo "deployment started"
              docker build --build-arg TMDB_V3_API_KEY=d5ca9ac9c910b2b310aa01e2d794a5e8 -t netflix .
              docker tag netflix:latest rutvikg/netflix:latest
              docker push rutvikg/netflix:latest
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
