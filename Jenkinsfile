pipeline {
	agent any
	stages {
		stage ('SCM Checkout') {
			steps {
				script {
					CI_ERROR = "Failed while checking out SCM"
				}
				sh """
					if [ -z  ${PACKAGE_NAME} ]; then
						exit 1
					fi
				"""
			}
		}
		stage ('Build Application') {
			steps {
				script {
					CI_ERROR = "Failed while building application"
				}
				sh """
					if [ -d ${PACKAGE_NAME} ]; then
						exit 1
					else
						mkdir ${PACKAGE_NAME}
						cd ${PACKAGE_NAME}
						sudo yum -y install ${PACKAGE_NAME}
					fi
				"""
			}
		}
		stage ('Deploy Application') {
			steps {
				script {
					CI_ERROR = "Failed while deploying application"
				}
				sh """
					if [ `which ${PACKAGE_NAME}` ]; then
						echo "${PACKAGE_NAME} installed"
					else
						echo "${PACKAGE_NAME} is not installed"
						exit 1
					fi
				"""		
			}
			
			
		}
	}
	
	post {
		always {
			script {
				CONSOLE_LOG = "${env.BUILD_URL}/console"
				BUILD_STATUS = currentBuild.currentResult
				if (currentBuild.currentResult == 'SUCCESS') {
					CI_ERROR = "NA"
					}
				}
				sh """
					TODAY=`date +"%b %d"`
					sed -i "s/%%JOBNAME%%/${env.JOB_NAME}/g" report.html
					sed -i "s/%%BUILDNO%%/${env.BUILD_NUMBER}/g" report.html
					sed -i "s/%%DATE%%/\${TODAY}/g" report.html
					sed -i "s/%%BUILD_STATUS%%/${BUILD_STATUS}/g" report.html
					sed -i "s/%%ERROR%%/${CI_ERROR}/g" report.html
					sed -i "s|%%CONSOLE_LOG%%|${CONSOLE_LOG}|g" report.html
				"""
				publishHTML(target:[
					allowMissing: true,
					alwaysLinkToLastBuild: true, 
					keepAll: true, 
					reportDir: "${WORKSPACE}", 
					reportFiles: 'report.html', 
					reportName: 'CI-Build-HTML-Report', 
					reportTitles: 'CI-Build-HTML-Report'
					])
				sendSlackNotifcation()
			}
		}
	}

def sendSlackNotifcation() 
{ 
	if ( currentBuild.currentResult == "SUCCESS" ) {
		buildSummary = "Job:  ${env.JOB_NAME}\n Status: *SUCCESS*\n Build Report: ${env.BUILD_URL}CI-Build-HTML-Report"

		slackSend color : "good", message: "${buildSummary}", channel: '#test-ci-alerts'
		}
	else {
		buildSummary = "Job:  ${env.JOB_NAME}\n Status: *FAILURE*\n Error description: *${CI_ERROR}* \nBuild Report :${env.BUILD_URL}CI-Build-HTML-Report"
		slackSend color : "danger", message: "${buildSummary}", channel: '#test-ci-alerts'
		}
}