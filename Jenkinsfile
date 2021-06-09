@Library("21re") _

gen.init()

def serviceName = "docker-mailserver"

reportSlack {
  node {
    def scmVars = checkout([
        $class: 'GitSCM',
        branches: [[name: 'refs/heads/'+env.BRANCH_NAME]],
        doGenerateSubmoduleConfigurations: false,
        extensions: [[$class: 'SubmoduleOption',
                        disableSubmodules: false,
                        parentCredentials: true,
                        recursiveSubmodules: true,
                        reference: '',
                        trackingSubmodules: false]], 
        submoduleCfg: [],         
        userRemoteConfigs: [[credentialsId: '2f2f8f0f-c144-482c-bd59-105a379b4ad7',
        url: 'git@github.com:21re/docker-mailserver.git']]])

    stage ("Build image") {
      sh """
      docker build --pull -t 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:${gen.VERSION} .
      """
    }

    stage ("Publish") {
      sh """
        set +x
        eval \$(aws ecr get-login --region eu-west-1 --no-include-email)
        set -x

        aws ecr describe-repositories --repository-names ${serviceName} || aws ecr create-repository --repository-name ${serviceName}
        docker push 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:${gen.VERSION}
      """
    }

    if(gen.deploy) {
      stage ("Mark latest") {

        sh """
          docker tag 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:${gen.VERSION} 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:latest
          docker push 351075005187.dkr.ecr.eu-west-1.amazonaws.com/docker-mailserver:latest
        """
      }
    }
  }
}
