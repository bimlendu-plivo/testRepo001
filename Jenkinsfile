#!groovy

@Library('plivo_standard_libs@ecs') _

globalDeployConfig = libraryResource 'globalDeployConfig.json'
globalDeployConfig = jsonParse(globalDeployConfig)

buildContext = [:]
buildContext << globalDeployConfig

pipeline {
	agent {
		label globalDeployConfig?.global?.defaults?.agentLabel
	}

	stages {
		stage('Checkout') {
			steps {
				deleteDir()
				checkout scm 

				checkout([
					$class: 'GitSCM',
					branches: scm.branches,
					doGenerateSubmoduleConfigurations: false,
					extensions: [[
						$class: 'SubmoduleOption',
						parentCredentials: true,
						recursiveSubmodules: true
					]], 
					userRemoteConfigs: scm.userRemoteConfigs
					])


				script {
					projectDeployConfig = readLocalJSON([
						file: 'deployConfig.json'
					])
					buildContext << projectDeployConfig

					String version = new java.text.SimpleDateFormat("yy.MM.dd.${currentBuild.id}").format(new Date())
					buildContext['version'] = version
					
					sh 'printenv'
				}
			}
		}

		stage('BuildDockerImage') {
			steps {
				buildDockerImage([
					buildContext: buildContext
				])
			}
		}
	}
}