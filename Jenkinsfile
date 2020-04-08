
node {
     try{
    // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
    def server = Artifactory.server "artifactory"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    
 rtMaven.tool = "maven"
	

    stage('Clone sources') {
        git url: 'https://github.com/erharshvsingh07/webapp.git'
    }
	
	
    stage('SonarQube Analysis') {
        withSonarQubeEnv(credentialsId: 'jenkinsonar', installationName: 'sonarqube') { 
       		sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://13.78.16.99:9000// -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java'
        }
	timeout(time: 1, unit: 'HOURS') { 
	    def qg = waitForQualityGate() 
	    if (qg.status != 'OK') {
	      error "Pipeline aborted due to quality gate failure: ${qg.status}"
	    }
	}             
  } 
	
	stage('Maven build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
    }
	
	stage('Deploy to QA') {
	deploy adapters: [tomcat7(credentialsId: 'qatomcat', path: '', url: 'http://18.223.239.239:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
	
    }
	
    stage('Artifactory configuration') {
        // Tool name from Jenkins configuration
        rtMaven.tool = "maven"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment.
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
    }

     stage('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage('UI QA') {
        buildInfo = rtMaven.run pom: 'functionaltest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
    	
    }
	
    stage('Deploy to Prod') {
	      deploy adapters: [tomcat7(credentialsId: 'AWStomcat', path: '', url: 'http://3.14.10.76:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
    }

    stage('Sanity Test') {
        buildInfo = rtMaven.run pom: 'Acceptancetest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
    }
	slackSend channel: 'project-dcs', message: "Build Completed", tokenCredentialId: 'slack'
}
 catch (exc) {
 	echo 'Build failed'
	slackSend channel: 'project-dcs', message: "Build Failed", tokenCredentialId: 'slack'
 }
}
	 
