@Library("21re") _

gen.init()

def serviceName = "docker-mailserver"

reportSlack {
  node {
    def scmVars = checkout scm

    stage ("Build image") {
      sh """
      docker build --pull -t 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:${VERSION} .
      """
    }

    stage ("Publish") {
      sh """
        set +x
        eval \$(aws ecr get-login --region eu-west-1 --no-include-email)
        set -x
projectName
        aws ecr describe-repositories --repository-names ${serviceName} || aws ecr create-repository --repository-name ${serviceName}
        docker push 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:${VERSION}
      """
    }

    if(gen.deploy) {
      stage ("Mark latest") {

        sh """
          docker tag 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:${VERSION} 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:latest
          docker push 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:latest
        """
      }
    } else {
        notifyGithub.call([gitRepo: scmVars.GIT_URL, gitCommit: scmVars.GIT_COMMIT])
    }
  }

  if(gen.deploy) {
    node {
      checkout scm

      // dockerMarkLatest([name: serviceName, version: gen.VERSION, withDatabase: true])

      serviceDeploySwarm([descriptor: "deployment-swarm.yaml", tag: gen.VERSION, stage: "testbed", numReplicas: 1])
    }

    // node {
    //   serviceSwitchTrafficSwarm([name: serviceName, tag: gen.VERSION, stage: "testbed"])
    // }

    node {
      serviceCleanupSwarm([name: serviceName, tag: gen.VERSION, stage: "testbed"])
    }

    // deployStaging([serviceName: serviceName, version: gen.VERSION, numReplicas: 1])

    // deployLive([serviceName: serviceName, version: gen.VERSION, numReplicas: 1])

  } else {
    serviceDeployRemove([name: serviceName, deployTimeout: 3, removeTimeout: 48, deployParameter:[descriptor: "deployment-swarm.yaml", tag: gen.VERSION, stage: "testbed", numReplicas: 1]])
  }
}
