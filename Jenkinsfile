pipeline{
  agent any
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
  }
}
