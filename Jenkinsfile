node {
  def projectPath = env.JOB_NAME.tokenize( '/' )
  def organization = projectPath[0]
  def projectName = projectPath[1]

  def rtcUrl='https://localhost:9443/ccm'
  def rtcUsername='pierodibello'
  def rtcPassword='123123123123'
  def remoteWorkspace="sync-workspace-$organization-$projectName-${env.BUILD_NUMBER}"
  def localWorkspace='local-sync-workspace'
  def gitCloneDirectory='git-repo'

  try {
    stage('Git clone') {
        dir('git-repo') {
          checkout([$class: 'GitSCM', branches: [[name: '*/*']], userRemoteConfigs: [[url: "https://github.com/$organization/${projectName}.git"]]])
          env.BRANCH_NAME = branchName()
       }
     }
     stage('RTC clone') {
        scm "login -u $rtcUsername -P $rtcPassword -r $rtcUrl"
        scm "create workspace -r $rtcUrl -s ${env.BRANCH_NAME} $remoteWorkspace"
        scm "load -r $rtcUrl -f -d $localWorkspace --all $remoteWorkspace"
     }
     stage('Sync Git to RTC') {
        sync gitCloneDirectory, "$localWorkspace/SRC/$projectName"
        scm "checkin --comment 'synch commit' $localWorkspace"
        scm "deliver -d $localWorkspace --source $remoteWorkspace -r $rtcUrl"
    }
  }
  catch(exception) {
    mailSyncFailed()
    throw exception
  }
  finally {
    deleteDir()
    scm "delete workspace -r $rtcUrl $remoteWorkspace"
  }
}

def scm(command) {
  sh "/opt/scmtools/eclipse/scm $command"
}

def sync(gitFolder, rtcFolder) {
  sh "rsync -av --progress $gitFolder/* $rtcFolder --exclude .git --delete"
}

def mailSyncFailed() {
   mail subject: "Sync to RTC failed in Jenkins: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: 'paolo.dincau@xpeppers.com',
        body: "Sync to RTC ${env.JOB_NAME} failed in Jenkins.\n\nSee ${env.BUILD_URL}"
}

def branchName() {
  def completeName = sh(script: 'git name-rev --name-only HEAD', returnStdout: true)
  def matcher = completeName =~ /remotes\/origin\/(.+)/;
  matcher[0][1];
}
