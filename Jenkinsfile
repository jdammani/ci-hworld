#!groovy


def buildStartTime = ""
def committerUserName = ""
def commitId = ""

def CleanUp(location){
	echo "Removing directory - " + location
	//bat "rmdir /s /Q $location"
}


def GetUserName (commitHash)
	{
		// Git committer User id
		echo "WorkSpacePath is ${WORKSPACE}"
		sh "cd ${WORKSPACE};git show -s --format='%an' ${commitHash} > commandResult"
		result = readFile('commandResult').trim()
		result = result.replace(" ", "").replace("," ,"")
		if (result.contains("\\"))
			{
			result = result.split("\\\\")[1]		
			}
		//echo "Git committer email: ${result}"
		return result

	}	
	

pipeline
	{
		agent { node { label 'test' } }
		//agent any
		stages
			{
			// clean up
			stage('cleanup at Beginning') 
				{
				steps
					{
						cleanWs()		 
					        //CleanUp('%workspace%')
					}
				}

			// Source code checkout
			stage('Checkout SourceCode') 
				{
				steps
					{					
					echo 'Pulling...' + env.BRANCH_NAME
					script 
						{						
							def now = new Date()
							buildStartTime = now.format("yyyy-MM-dd'T'HH:mm:ss.SSS+00:00", TimeZone.getTimeZone("UTC")) 
							println "starttime:" + buildStartTime
																
							commitId = checkout(scm).GIT_COMMIT //checkout scm
							echo 'Commit id is : ' + commitId

							committerUserName = GetUserName(commitId)
							echo 'UserName:'+committerUserName
						}										
					}
				}							

			// build
			stage('Build') 
				{
				steps
					{
					script 
						{		
                           				echo "executing ${WORKSPACE}/src/build-hworld.sh"
							sh "echo $PWD"
							sh "cd ${WORKSPACE}/src/;bash ./build-hworld.sh ${WORKSPACE}/src/"
							echo "completed build"													
						}
					}
				}
				
			// upload
			stage('Upload to S3 repository') 
				{
				steps
					{
					script 
						{		
                           				echo "uploading output.tar.gz to cicd-buildpoc-repo/repository/cihworld/1.0/${env.BUILD_NUMBER}/"
							sh "aws s3 cp ${WORKSPACE}/build/output.tar.gz s3://cicd-buildpoc-repo/repository/cihworld/1.0/${env.BUILD_NUMBER}/output.tar.gz"
							echo "upload successful"													
						}
					}
				}				
				
				
			}    	
	}					
