pipeline {
	environment {
    		def APP_NAME = "sample"
    		def GIT_REPO_NAME = "mrganeshkudale"
    		def DEPLOY_ENV = "dev"
	}
    	agent any
    	stages {
		stage('Code Checkout') {
			steps {
				sh "if [ -d ${APP_NAME} ]; then rm -rf ${APP_NAME}; fi"
				sh "git clone https://github.com/${GIT_REPO_NAME}/${APP_NAME}.git"
				script{
					/** @return The tag name, or `null` if the current commit isn't a tag. */
					String gitTagName() {
					    commit = getCommit()
					    if (commit) {
						desc = sh(script: "git describe --tags ${commit}", returnStdout: true)?.trim()
						if (isTag(desc)) {
						    return desc
						}
					    }
					    return null
					}

					/** @return The tag message, or `null` if the current commit isn't a tag. */
					String gitTagMessage() {
					    name = gitTagName()
					    msg = sh(script: "git tag -n10000 -l ${name}", returnStdout: true)?.trim()
					    if (msg) {
						return msg.substring(name.size()+1, msg.size())
					    }
					    return null
					}

					String getCommit() {
					    return sh(script: 'git rev-parse HEAD', returnStdout: true)?.trim()
					}

					@NonCPS
					boolean isTag(String desc) {
					    match = desc =~ /.+-[0-9]+-g[0-9A-Fa-f]{6,}$/
					    result = !match
					    match = null // prevent serialisation
					    return result
					}
					
					    GIT_TAG_NAME = gitTagName()
    					    GIT_TAG_MESSAGE = gitTagMessage()
						sh "echo ${GIT_TAG_NAME}"
						sh "echo ${GIT_TAG_MESSAGE}"
				}
			}
		}
		stage('Azure Cloud Connect'){
			steps {
				sh "az login --identity"
				sh "az account set --subscription xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
				sh "az aks get-credentials --resource-group atos-tra-pla-rg --name atos-tra-pla-cluster"			
			}
		}
		stage('Build & Image'){
			steps {
				sh "az acr build -r tntaksreg -t ${APP_NAME} ."			
			}
		}
		stage('Deploy'){
			steps {
				sh "kubectl delete deployment ${APP_NAME}-deployment --namespace=${DEPLOY_ENV}"
				sh "kubectl apply -f ${DEPLOY_ENV}.yml --namespace=${DEPLOY_ENV}"
			}
		}
    	}
	post { 
		success { 
		    echo "Your application URL will be - http://${APP_NAME}.e46708b92c054086909b.eastus.aksapp.io"
		}
		failure { 
		    echo "Please check logs for more details."
		}
    	}
}
