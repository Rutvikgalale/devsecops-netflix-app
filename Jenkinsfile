pipeline{
  agent any
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
  }
}
