#!groovy
def uniquename = { String prefix ->
  sh "cat /dev/urandom | tr -dc 'a-z0-9' | fold -w 16 | head -n 1 > suffix"
  suffix = readFile("suffix").trim()
  return prefix + suffix
}

node {
  stage 'Create Project'
  project = uniquename('maps-app-')
  echo "Creating new project ${project}"
  sh "oc new-project ${project}"

  stage 'Create NationalParks back-end'
  def nationalParksURL = "https://github.com/csrwng/nationalparks.git"
  def nationalParksBranch = "Jenkinsfile"
  checkout([$class: "GitSCM", branches: [[name: "*/${nationalParksBranch}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "nationalparks"]], submoduleCfg: [], userRemoteConfigs: [[url: "${nationalParksURL}"]]])
  sh "oc new-app -f nationalparks/ose3/pipeline-buildconfig-template.json -p GIT_URI=${nationalParksURL} -p GIT_REF=${nationalParksBranch} -n ${project}"

  stage 'Create MLBParks back-end'
  def mlbParksURL = "https://github.com/csrwng/mlbparks.git"
  def mlbParksBranch = "Jenkinsfile"
  checkout([$class: "GitSCM", branches: [[name: "*/${mlbParksBranch}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "mlbparks"]], submoduleCfg: [], userRemoteConfigs: [[url: "${mlbParksURL}"]]])
  sh "oc new-app -f mlbparks/ose3/pipeline-buildconfig-template.json -p GIT_URI=${mlbParksURL} -p GIT_REF=${mlbParksBranch} -n ${project}"

  stage 'Create ParksMap front-end'
  def parksMapURL = "https://github.com/csrwng/parksmap-web.git"
  def parksMapBranch = "Jenkinsfile"
  checkout([$class: "GitSCM", branches: [[name: "*/${parksMapBranch}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "parksmap"]], submoduleCfg: [], userRemoteConfigs: [[url: "${parksMapURL}"]]])
  sh "oc new-app -f parksmap/ose3/pipeline-buildconfig-template.json -p GIT_URI=${parksMapURL} -p GIT_REF=${parksMapBranch} -n ${project}"

  stage 'Make project accessible'
  sh "oc policy add-role-to-user admin developer -n ${project}"
}

stage 'Build Back-ends'
parallel (
  "nationalparks": { 
    node {
      // TODO: switch to using openshiftBuild when openshiftBuild DSL step supports Jenkins strategy
      // openshiftBuild buildConfig: "nationalparks-pipeline", namespace: project, verbose: true
      sh 'oc start-build nationalparks-pipeline --wait'
    }
  },
  "mlbparks": {
    node {
      // TODO: switch to using openshiftBuild when openshiftBuild DSL step supports Jenkins strategy
      sh 'oc start-build mlbparks-pipeline --wait'
    }
  }
)

node {
  stage 'Build Front-end' 
  // TODO: switch to using openshiftBuild when openshiftBuild DSL step supports Jenkins strategy
  // openshiftBuild buildConfig: "parksmap-pipeline", namespace: project, verbose: true
  sh 'oc start-build parksmap-pipeline --wait'
}

node {
  stage 'Approval'
  sh "oc get route parksmap -o jsonpath='{ .spec.host }' > routehost"
  routehost = readFile("routehost").trim()
  input "Parksmap is running at http://${routehost}. Continue?"

  stage 'Cleanup'
  sh "oc delete project ${project}"
}
