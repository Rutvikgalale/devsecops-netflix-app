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
    stage("owasp dependency check"){
      steps{
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
  }
}
