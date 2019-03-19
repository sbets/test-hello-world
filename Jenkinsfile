#!groovy

@Library('utils') import com.firm.*

def credentialsId = "dockerhub"
def registryUrl = "docker.io"

def registry = new DockerRegistryCredentials()
def registryLogin = new DockerRegistryLogin()
def building = new Images(this)

node {
  log.info 'Starting...'
  log.info 'Getting credentials for DockerRegistry...'
  
  stage('Get Docker Registry credentials and Login to') {
      password = registry.getPassword(credentialsId)
      username = registry.getUsername(credentialsId)
      
      login = registryLogin.loginToDockerRegistry(registryUrl, username, password)
      sh "echo ${login}"
  }
  log.info 'Building docker image...'
  
  stage('Fetch code and build an image') {
      checkout([$class: 'GitSCM', 
                branches: [[name: 'infra/spinnaker']], 
                userRemoteConfigs: [[url: 'https://github.com/serhii-dog/test-hello-world.git']]])
      
      building.buildAnImage("serhii2dog/test", "${params.ReleaseVersion}", envVars:'${params.EnvVars}', buildArgs: '${params.BuildArgs}')
  }
  log.info 'Pushing docker image...'
  
  stage('Push docker image to registry') {
      building.pushAnImage("serhii2dog/test", "${params.ReleaseVersion}")
  }
  
}
