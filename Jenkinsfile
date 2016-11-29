node 
{
   
    String jobName = "${env.JOB_NAME}"
	if( jobName == "adc-pipeline-scipt" )
	{
		// Build using a custom docker container
		def buildContainer = docker.image('mvn-npm:latest')
    
		buildContainer.inside('-v /m2repo:/m2repo') 
		{
        
			// Set up a shared Maven repo so we don't need to download all dependencies on every build.
			//writeFile file: 'settings.xml', text: '<settings><localRepository>/m2repo</localRepository></settings>'
      
     
			stage 'Stage 1:Checking out GitHub Repo'
			git url: 'https://github.com/epasham/Checklist.git'
      
			stage 'Stage 2:Run npm update'
			sh 'npm update'
      
			stage 'Stage 3:Run bower update'
			sh 'bower update --allow-root'
      
			stage 'Stage 4:Run npm test'
			sh 'npm test'
      
			//stage 'Stage 5: Run gradle scan'
			//gradle findbugsMain checkstyleMain checkThePMD fortifyReport
		  
			stage 'Stage 5:Run mvn goals'
			sh 'mvn -Dmaven.test.skip=true compile package' 
			// Build with maven settings.xml file that specs the local Maven repo.
			sh 'echo $(pwd)'
			stash includes: './target/*.war,Dockerfile', name: 'dockerBuild'
		  
			stage 'Stage 6:Static Code Analysis'
			step([$class: 'CheckStylePublisher', pattern: 'target/reports/checkstyle/*.xml'])
			step([$class: 'FindBugsPublisher', pattern: 'target/reports/findbugs/*.xml'])
			step([$class: 'PmdPublisher', pattern: 'target/reports/pmd.xml'])
		  
			stage 'Stage 7:Archive Artifacts'
			step([$class: 'ArtifactArchiver', artifacts: 'target/*.jar, target/*.war', fingerprint: true])

			//stage 'Stage 8:Unit Tests'
			//step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'])

			stage 'Stage 9:Code Coverage'
			step([$class: 'JacocoPublisher', execPattern: '**/target/**.exec', classPattern: '**/classes', sourcePattern: '**/src/main/java'])

			//stage 'Stage 10:White Source'
			//step([$class: 'WhiteSourcePublisher', jobCheckPolicies: 'global', jobApiToken: 'f350b639-e7cc-447b-a830-3737eee4a6c0', product: 'CICD-builder', productVersion: '1.0.0', projectToken: '""', libIncludes: '""', libExcludes: '""', mavenProjectToken: '""', requesterEmail: 'cicd-dev-team@pwc.com', moduleTokens: '""', modulesToInclude: '""', modulesToExclude: '""', ignorePomModules: 'false'])

			stage 'Stage 11:HPE Security Fortify Scan'
			step([$class: 'FodBuilder', filePattern: '""', assessmentTypeId: '170', applicationName: 'AUS-IFS-Enterprise API', releaseName: 'V1.0', technologyStack: 'JAVA/J2EE', languageLevel: '1.8', runOpenSourceAnalysis: false, isExpressScan: false, isExpressAudit: false, doPollFortify: false, doPrettyLogOutput: false, includeAllFiles: false, includeThirdParty: false])

			//stage 'Stage 12:Black Duck Scan'
			//step([$class: 'PostBuildHubScan', scans: '{}', hubProjectName: 'CICD-builder', hubVersionPhase: 'In Development', hubVersionDist: 'External', hubProjectVersion: '1.0.0', scanMemory: '4096', shouldGenerateHubReport: false, bomUpdateMaxiumWaitTime: '5', dryRun: false, verbose: false])
			//hub_scan bomUpdateMaxiumWaitTime: '5', dryRun: false, hubProjectName: 'Checklist', hubProjectVersion: '1.0.0', hubVersionDist: 'EXTERNAL', hubVersionPhase: 'PLANNING', scanMemory: '4096', scans: [[scanTarget: '']], shouldGenerateHubReport: false
		}  
  
      stage 'Stage 13:Build docker image'
      unstash 'dockerBuild'
      // Build final image using the Dockerfile and the docker.build command
      // This container only contains the packaged war, not the source or interim build steps
      def adcImg = docker.build("ekambaram/checklist:${env.BUILD_TAG}")
   
      registry_url = 'https://index.docker.io/v1/' // Docker Hub
      
      stage 'Stage 14: Publish to DockerHub'
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker_creds_id',usernameVariable: 'USERNAME',passwordVariable: 'PASSWORD']]) {
      echo "${env.USERNAME} ${env.PASSWORD}"
      sh "docker login -u $env.USERNAME -p $env.PASSWORD $registry_url"
      sh "docker push ekambaram/checklist:${env.BUILD_TAG}"
      }
	  
	}
	if( jobName == "adc-pipeline-night" )
	{
		// Build using a custom docker container
		def buildContainer = docker.image('mvn-npm:latest')
    
		buildContainer.inside('-v /m2repo:/m2repo') 
		{
        
			// Set up a shared Maven repo so we don't need to download all dependencies on every build.
			//writeFile file: 'settings.xml', text: '<settings><localRepository>/m2repo</localRepository></settings>'
      
     
			stage 'Stage 1:Checking out GitHub Repo'
			git url: 'https://github.com/epasham/Checklist.git'
      
			stage 'Stage 2:Run npm update'
			sh 'npm update'
		}
	}	
}
