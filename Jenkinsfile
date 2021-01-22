properties([
  parameters([
      string(name: 'DOCKER_REGISTRY_DOWNLOAD_URL', defaultValue: 'nexus-docker-private-group.ossim.io', description: 'Repository of docker images')
  ]),
  pipelineTriggers([[$class: "GitHubPushTrigger"]]),
  [$class: 'GithubProjectProperty', displayName: '', projectUrlStr: 'https://github.com/ossimlabs/omar-hibernate-spatial'],
  buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '3', daysToKeepStr: '', numToKeepStr: '20')),
  disableConcurrentBuilds()
])

podTemplate() {
  node(POD_LABEL) {
    stage("Checkout branch") {
      scmVars = checkout(scm)

      GIT_BRANCH_NAME = scmVars.GIT_BRANCH
      BRANCH_NAME = """${sh(returnStdout: true, script: "echo ${GIT_BRANCH_NAME} | awk -F'/' '{print \$2}'").trim()}"""
      GRADLE_BUILD_VERSION = sh(returnStdout: true, script: "grep buildVersion gradle.properties").trim()

      script {
        if (BRANCH_NAME == 'master') {
          buildName "${GRADLE_BUILD_VERSION} - ${BRANCH_NAME}"
        } else {
          buildName "${GRADLE_BUILD_VERSION} - ${BRANCH_NAME}-SNAPSHOT"
        }
      }
    }

    stage("Load Variables") {
      withCredentials([string(credentialsId: 'o2-artifact-project', variable: 'o2ArtifactProject')]) {
        step([$class     : "CopyArtifact",
              projectName: o2ArtifactProject,
              filter     : "common-variables.groovy",
              flatten    : true])
      }

      load "common-variables.groovy"
    }

    stage("Assemble") {
      sh """
        ./gradlew assemble -PossimMavenProxy=${MAVEN_DOWNLOAD_URL}
        """
      archiveArtifacts "plugins/*/build/libs/*.jar"
    }

    stage("Publish Nexus") {
      withCredentials([[$class          : 'UsernamePasswordMultiBinding',
                        credentialsId   : 'nexusCredentials',
                        usernameVariable: 'MAVEN_REPO_USERNAME',
                        passwordVariable: 'MAVEN_REPO_PASSWORD']]) {
        sh """
          ./gradlew publish -PossimMavenProxy=${MAVEN_DOWNLOAD_URL}
        """
      }
    }
  }
}