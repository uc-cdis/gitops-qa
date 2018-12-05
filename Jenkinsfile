#!groovy

@Library('cdis-jenkins-lib@refactor/microservices') _

runPipeline {
 pipeline = 'gitops'

 namespaces = [
   "jenkins-brain",
   "jenkins-niaid",
   "jenkins-dcp"
 ]

 skipDeploy = 'true'
}