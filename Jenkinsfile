#!groovy

pipeline {
  agent any

  stages {
    stage('FetchCode') {
      steps {
        dir('cdis-manifest') {
          checkout scm
        }
        dir('gen3-qa') {
          git(
            url: 'https://github.com/uc-cdis/gen3-qa.git',
            branch: 'master'
          )
        }
        dir('cloud-automation') {
          git(
            url: 'https://github.com/uc-cdis/cloud-automation.git',
            branch: 'master'
          )
          script {
            env.GEN3_HOME = env.WORKSPACE + "/cloud-automation"
          }
        }
      }
    }
    stage('DetectChanges') {
      steps {
        script {
          def changeLogSets = currentBuild.changeSets
          def foundMatch=false;
          for (int i = 0; !foundMatch && i < changeLogSets.size(); i++) {
            def entries = changeLogSets[i].items
            for (int j = 0; !foundMatch && j < entries.length; j++) {
              def affectedPaths = entries[j].getAffectedPaths()
              if (affectedPaths.contains('qa-brain/manifest.json')) {
                env.AFFECTED_PATH = 'qa-brain/manifest.json'
                env.KUBECTL_NAMESPACE = 'jenkins-brain'
                env.COPY_MANIFEST = 'true';
                foundMatch = true
              } else if (affectedPaths.contains('qa-niaid/manifest.json')) {
                env.AFFECTED_PATH = 'qa-niaid/manifest.json'
                env.KUBECTL_NAMESPACE = 'jenkins-niaid'
                env.COPY_MANIFEST = 'true';
                foundMatch = true
              } else if (affectedPaths.contains('jenkins-brain/manifest.json')) {
                env.AFFECTED_PATH = 'jenkins-brain/manifest.json'
                env.KUBECTL_NAMESPACE = 'jenkins-brain'
                foundMatch = true
              } else if (affectedPaths.contains('jenkins-niaid/manifest.json')) {
                env.AFFECTED_PATH = 'jenkins-niaid/manifest.json'
                env.KUBECTL_NAMESPACE = 'jenkins-niaid'
                foundMatch = true
              } else {
                println "ignoring git changes to: " + affectedPaths.join("\n")
              }
            }
          }
          if (!foundMatch) {
            println "testable stuff was not affected, aborting"
            env.ABORT_SUCCESS = 'true';
            env.COMPY_MANIFEST = 'false';
            currentBuild.result = 'SUCCESS'
          }
        }
      }
    }
    stage('SubstituteManifest') {
      when {
        environment name: 'COPY_MANIFEST', value: 'true'
      }
      steps {
        script {
          dirname = sh(script: "kubectl -n $env.KUBECTL_NAMESPACE get configmap global -o jsonpath='{.data.hostname}'", returnStdout: true)
        }
        dir('cdis-manifest') {
          withEnv(["fromPath=$env.AFFECTED_PATH", "toPath=$dirname/manifest.json"]) {
            sh "cp $env.fromPath $env.toPath"
          }
        }    
      }
    }
    stage('K8sDeploy') {
      when {
        environment name: 'ABORT_SUCCESS', value: 'false'
      }
      steps {
        withEnv(['GEN3_NOPROXY=true', "vpc_name=$env.KUBECTL_NAMESPACE", "GEN3_HOME=$env.WORKSPACE/cloud-automation"]) {
          echo "GEN3_HOME is $env.GEN3_HOME"
          echo "GIT_BRANCH is $env.GIT_BRANCH"
          echo "GIT_COMMIT is $env.GIT_COMMIT"
          echo "KUBECTL_NAMESPACE is $env.KUBECTL_NAMESPACE"
          echo "WORKSPACE is $env.WORKSPACE"
          script {
            uid = BUILD_TAG.replaceAll(' ', '_').replaceAll('%2F', '_')
            sh("bash cloud-automation/gen3/bin/klock.sh lock jenkins "+uid)
          }
          sh "bash cloud-automation/gen3/bin/kube-roll-all.sh"
          sh "bash cloud-automation/gen3/bin/kube-wait4-pods.sh || true"
        }
      }
    }
    stage('RunTests') {
      when {
        environment name: 'ABORT_SUCCESS', value: 'false'
      }
      steps {
        dir('gen3-qa') {
          withEnv(['GEN3_NOPROXY=true', "vpc_name=$env.KUBECTL_NAMESPACE", "GEN3_HOME=$env.WORKSPACE/cloud-automation", "NAMESPACE=$env.KUBECTL_NAMESPACE", "TEST_DATA_PATH=$env.WORKSPACE/testData/"]) {
            sh "bash ./jenkins-simulate-data.sh $env.KUBECTL_NAMESPACE"
            sh "bash ./run-tests.sh $env.KUBECTL_NAMESPACE"
          }
        }
      }
    }
  }
    
  post {
    success {
      echo "https://jenkins.planx-pla.net/ $env.JOB_NAME pipeline succeeded"
    }
    failure {
      echo "Failure!"
      archiveArtifacts artifacts: '**/output/*.png', fingerprint: true
      junit "gen3-qa/output/*.xml"
      //slackSend color: 'bad', message: "https://jenkins.planx-pla.net $env.JOB_NAME pipeline failed"
    }
    unstable {
      echo "Unstable!"
      //slackSend color: 'bad', message: "https://jenkins.planx-pla.net $env.JOB_NAME pipeline unstable"
    }
    always {
      echo "done"
      script {
        if (env.ABORT_SUCCESS == 'false') {
          uid = BUILD_TAG.replaceAll(' ', '_').replaceAll('%2F', '_')
          sh("bash cloud-automation/gen3/bin/klock.sh unlock jenkins "+uid)
          junit("gen3-qa/output/*.xml")
        }
      }
    }
  }
}
