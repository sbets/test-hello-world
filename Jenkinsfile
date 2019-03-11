#!groovy

@Library('utils') import com.firm.*

def credentialsId = "dockerhub"
def registryUrl = "docker.io"

def utils = new Utilities(this)
def registry = new DockerRegistryCredentials()
def registryLogin = new DockerRegistryLogin()
def building = new Images(this)
def k8sCluster = new KubernetesInitConnection(this)
def helm = new HelmInit(this)
def secret = new KubeSecret(this)

node {
  log.info 'Starting'
  
  stage('Get list of system environment variables') {
      utils.mvn('env')
  }
  
  stage('Get Docker Registry credentials and Login to') {
      password = registry.getPassword(credentialsId)
      username = registry.getUsername(credentialsId)
      
      login = registryLogin.loginToDockerRegistry(registryUrl, username, password)
      sh "echo ${login}"
  }
  
  stage('Fetch code and build an image') {
      checkout([$class: 'GitSCM', 
                branches: [[name: '*/master']], 
                userRemoteConfigs: [[url: 'https://github.com/serhii-dog/test-hello-world.git']]])
      
      building.buildAnImage("serhii2dog/test", "${params.ReleaseVersion}", envVars:'${params.EnvVars}', buildArgs: '${params.BuildArgs}')
  }
  
  stage('Push docker image to registry') {
      building.pushAnImage("serhii2dog/test", "${params.ReleaseVersion}")
  }
  
  stage('Init connection to K8S') {
      k8sCluster.initGoogleKubernetesConnection("lrcwebsite", "k8s-cs-pipeline")
  }
  
  stage('Create secrets') {
      secret.addPrivateRegistryCredentials(registryUrl, credentialsId)
  }
  
  stage('Init Helm') {
      helm.initHelmCli()
  }
  
  stage('Deploy application') {
      checkout([$class: 'GitSCM', 
                branches: [[name: 'infra/basic']], 
                userRemoteConfigs: [[url: 'https://github.com/serhii-dog/hello-world-helm.git']]])
      
      helm.deployHelmChart("hello-world-helm", "${params.ReleaseVersion}")
  }
}
