#!/usr/bin/env groovy
properties([
  buildDiscarder(logRotator(daysToKeepStr: '40', numToKeepStr: '40')),
  parameters([
    string(description: 'Git Branch Name', name: 'GIT_BRANCH', defaultValue: 'feature/jenkins'),
  ])
])
def podDefinition = """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: aat5-runner
    image: 'registry.hub.docker.com/library/node:stretch'
    tty: true
    command:
    - 'cat'
    env:
    - name: "GIT_SSL_NO_VERIFY"
      value: "true"
    resources:
      limits:
        memory: "1000Mi"
        cpu: "500m"
      requests:
        memory: "500Mi"
        cpu: "200m"
  nodeSelector:
    env: "prod"
  restartPolicy: "Never"
  securityContext: {}
"""
def connectionParam = [
  CONNECTION_USER: "svc_aat5@corp.emirates.com",
  WORKSPACE:  "~",
  HOSTNAME: "",
  BECOME_USER: "",
  CONNECTION_KEY: "FLX-devops-user"
]
podTemplate(yaml: podDefinition) {
timeout(120) {
  node(POD_LABEL) {
    timestamps {
      ansiColor('xterm') {
        try {
            stage('Checkout') {
              container('aat5-runner') {
                print "======== Checkout ========"
                git credentialsId: "ssh-auth", url: "ssh://git@git.emirates.com/flxb2b/ndc_sanity_pack.git", branch: "${params.GIT_BRANCH}"
              }
            }
            stage('Install Dependencies') {
              container('aat5-runner') {
                print "======== Installing Dependencies ========"
                sh "apt update && apt install -y jq"
                sh "npm install -g csvtojson"
                sh "npm install"
              }
            }
            stage('Run Sanity Pack') {
              container('aat5-runner') {
                print "======== RUN SANITY Pack ========"
                sh "npm run sanity"
              }
            }
        } catch(err) {
		
          office365ConnectorSend (
              webhookUrl: "",
              // message: "Build #${env.BUILD_NUMBER} (<${env.BUILD_URL}>)",
              message: "Error: ${env.newman_errors}",
              status: "Failed NDC Sanity TESTS"
          )
          echo "FAILURE - please check the log"
          throw err
		  
        } finally {
            junit 'newman/*.xml'
            archiveArtifacts artifacts: 'newman/**', onlyIfSuccessful: false
            stage('Archive Results') {
              container('aat5-runner') {
                print "======== Archiving Results ========"
              }
            }
        }
      }
    }
  }
}
}
