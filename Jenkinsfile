node {
	def jobName = env.JOB_NAME
	def projectName = jobName.split('/')[0]
	def mvn_version = 'M3'
	def platform = ['onprem']		// list of platform i.e. onprem or azcloud or both
	
	try {
	
		hipchatSend(color: "YELLOW", 
			message: "STARTED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}]")
		
		stage ('Preparation') {
			println "Build: " + projectName + " for branch " + env.BRANCH_NAME
			git(
				url: "https://github.com/raviteja0422/${projectName}.git",
				credentialsId: 'EAIGitHub',
				branch: env.BRANCH_NAME
			)		
			sh 'git config credential.helper cache'
			sh 'git config push.default simple'
			sh 'git clean -f'
			sh 'git reset --hard'
			sh 'git checkout .'
		}
		stage ('Build') {
			if (fileExists("${projectName}")) {
				if (env.BRANCH_NAME=='Integrations') {
					echo 'build Integrations branch'
	
					withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
						echo "${tool mvn_version}"
						def pom = readMavenPom file: "${projectName}/pom.xml"
						def versionList = pom.version.replace("-SNAPSHOT", "").tokenize(".")
						def version = "${versionList[0]}.${versionList[1]}.${versionList[2]}"
						dir ("${projectName}") {
							sh 'mvn clean package -Dmule.env=dev'
							dir ("target") {
								for (p in platform) {
									sh "/mule/scripts/deploy.sh push Development ${projectName} ${p} ${version}"
								}
							}
						}
					}
	
				} else if (env.BRANCH_NAME=='master') {
					echo 'build master branch'
				
					withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
						def pom = readMavenPom file: "${projectName}/pom.xml"
						def versionList = pom.version.replace("-SNAPSHOT", "").tokenize(".")
						def newRelease = Eval.me(versionList[2])+1
						echo "${newRelease}"
						def version = "${versionList[0]}.${versionList[1]}.${versionList[2]}"
						def newversion="${versionList[0]}.${versionList[1]}.${newRelease}-SNAPSHOT"
						echo "release version: ${version}"
						echo "new version: ${newversion}"
						dir ("${projectName}") {
							echo "release version: ${version}"
							sh "mvn clean versions:set -DnewVersion=${version}"
							sh 'mvn clean deploy -Dmule.env=stg'
							dir ('target') {
								for (p in platform) {
									sh "/mule/scripts/deploy.sh push Stage ${projectName} ${p} ${version}"
								}
								sh "cp ${projectName}-${version}.zip /jenkins/release"
							}
						}
						sh "git tag -f ${projectName}-${version} -m JenkinsRelease-${version}"
						sh 'git push -u origin master --tag'
						sh 'git checkout .'
						sh 'git checkout Integrations'
						sh 'git clean -f'
						sh 'git reset --hard'
						sh 'git checkout .'
						sh 'git pull origin Integrations'
						dir ("${projectName}") {
							sh "mvn clean versions:set -DnewVersion=${newversion}"
							sh 'git add -- pom.xml'
						}
						sh "git commit -m jenkinsChangingVersion-${newversion}"
						sh 'git push -u origin Integrations'		
					}
	
				}
			} else {
				echo "${projectName} does not exist in repository"
				echo "No build is done"
			}				
		}
		hipchatSend(color: "GREEN", 
			message: "<img src='https://dujrsrsgsd3nh.cloudfront.net/img/emoticons/successful-1414026553.png'/> SUCCESSFUL Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] (<a href='${env.BUILD_URL}/console'>Details</a>)")
		
	} catch (e) {
		hipchatSend(color: "RED", 
			message: "<img src='https://dujrsrsgsd3nh.cloudfront.net/img/emoticons/failed-1414026532.png'/> FAILED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] (<a href='${env.BUILD_URL}/console'>Details</a>)")
		
		throw e;
	}
}
