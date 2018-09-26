node('master') {
  try {
    def server
    def buildInfo
    def rtMaven
    def app

    env.APP_NAME="aleph-formatter"
    env.AWS_REGION="eu-west-2"
    env.REGISTRY_URL="https://067460451267.dkr.ecr.eu-west-2.amazonaws.com/"
    env.TAG_NAME = "067460451267.dkr.ecr.eu-west-2.amazonaws.com/${env.APP_NAME}:${env.BUILD_NUMBER}"

    stage('SCM Checkout') {
        checkout scm
    }

    stage ('Artifactory configuration') {
        // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
        server = Artifactory.server 'Artifactory'

        rtMaven = Artifactory.newMavenBuild()
        rtMaven.tool = 'Default Maven' // Tool name defined in Jenkins configuration
        rtMaven.deployer releaseRepo: 'senapt-release-local', snapshotRepo: 'senapt-snapshot-local', server: server
    }

    stage('Build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -DskipTests=true'
    }
  } catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
}
def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
