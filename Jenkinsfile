
pipeline {
	agent any
	options {
		buildDiscarder( logRotator(numToKeepStr:'20', artifactNumToKeepStr: '1'))
		disableConcurrentBuilds()
		timeout(time: 5, unit: 'HOURS')   // timeout on whole pipeline job
        ansiColor('xterm')
	}
    tools {
        maven 'apache-maven-latest'
        jdk 'open-jdk-11'
    }
    stages {
	    stage('Preparation') {
		    steps {	
				echo 'Content of the workspace'
				sh "pwd"
				sh "ls"
				sh "cat .gitmodules"
				sh "git submodule update --init"
				sh "chmod 777 ./gemoc-studio/dev_support/jenkins/showGitBranches.sh"
				sh "./gemoc-studio/dev_support/jenkins/showGitBranches.sh ."
				// install JAVAFX TODO find a better way !
				// 
				sh '''
				   if [ ! -d ${HOME}/tools/javafx-sdk-11.0.2 ]; then 
				      wget https://gluonhq.com/download/javafx-11-0-2-sdk-linux  -O ${HOME}/tools/javafx-11-0-2-sdk-linux.zip 
				      unzip -n ${HOME}/tools/javafx-11-0-2-sdk-linux.zip -d ${HOME}/tools 
				      rm -f ${HOME}/tools/javafx-11-0-2-sdk-linux.zip 
				   else 
				      echo ${HOME}/tools/javafx-sdk-11.0.2 allready installed
				   fi
				'''
				//JAVAFX_HOME=${HOME}/javafx-sdk-11.0.2
			}
		}
		stage('Generate protocols') {
			steps { 
				nodejs(nodeJSInstallationName: 'nodejs-17.x') {
			    	dir ('gemoc-studio-modeldebugging/protocols/generators/ts/JSONSchema2APIProtocolGenerator') {
						sh 'npm config ls'
						sh 'npm install'
						sh 'npm run build'
						sh 'npm run generate'
					}
		        }
			}
	 	}
		stage('Pomfirst Build') {
			steps { 
				script {	
					withEnv(["JAVAFX_HOME=${HOME}/tools/javafx-sdk-11.0.2",
						     "MAVEN_OPTS=-Xmx2000m -XshowSettings:vm"]){
						dir ('gemoc-studio/dev_support/pomfirst_full_compilation') {
							sh "mvn -Dmaven.test.failure.ignore clean install --errors --show-version -Dstyle.color=always"
						}      
					}
				}
			}
	 	}
	 	/* stage('Update targetplatform') {
			steps { 
				script {	
					dir ('gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.targetplatform') {
						sh "./update_target.sh"
					} 
				}
			}
	 	} */
	 	stage('Tycho Build and unit tests') {
			steps { 
				script {	
					def studioVariant
					if(  env.JENKINS_URL.contains("https://ci.eclipse.org/gemoc/")){
						studioVariant = "Official build"
					} else {
						studioVariant = "${JENKINS_URL}"
					}
					// Run the maven build and unit tests only  
					// maven requires some ram to build the update site and product
					withEnv(["STUDIO_VARIANT=${studioVariant}","BRANCH_VARIANT=${BRANCH_NAME}", "JAVAFX_HOME=${HOME}/tools/javafx-sdk-11.0.2",
						"MAVEN_OPTS=-Xmx2000m -XshowSettings:vm"]){
						dir ('gemoc-studio/dev_support/tycho_full_compilation') {
							sh 'printenv'         
							sh "mvn -Dmaven.test.failure.ignore \"-Dstudio.variant=${studioVariant}\" -Dbranch.variant=${BRANCH_VARIANT} \
									-Djava.awt.headless=true \
									--projects !../../gemoc_studio/tests/org.eclipse.gemoc.studio.tests.system.lwb,!../../gemoc_studio/tests/org.eclipse.gemoc.studio.tests.system.mwb\
									clean install --errors --show-version  -Dstyle.color=always"
						}      
					}
				}
			}
			post {
				always {
					junit '**/target/surefire-reports/TEST-*.xml' 
				}
			}
	 	}
	 	stage('Tycho System tests') {
			steps { 
				script {	
					def studioVariant
					if(  env.JENKINS_URL.contains("https://hudson.eclipse.org/gemoc/")){
						studioVariant = "Official build"
					} else {
						studioVariant = "${JENKINS_URL}"
					}
					// Run the maven system tests only  
					// use less RAM to maven in order to give more to the UI test JVM
					withEnv(["STUDIO_VARIANT=${studioVariant}","BRANCH_VARIANT=${BRANCH_NAME}",
						"MAVEN_OPTS=-XshowSettings:vm"]){
						dir ('gemoc-studio/dev_support/tycho_full_compilation') {         
							wrap([$class: 'Xvnc', takeScreenshot: false, useXauthority: true]) {
								sh 'printenv'
								
								// use linux timeout in order to continue the video capture
								// capture the returnStatus in order to make sure to stop ffmepg even if an error occured
								def status = sh(returnStatus: true, script: "timeout -s KILL 60m \
									mvn -Dmaven.test.failure.ignore \"-Dstudio.variant=${studioVariant}\" -Dbranch.variant=${BRANCH_VARIANT} \
										--projects ../../gemoc_studio/tests/org.eclipse.gemoc.studio.tests.system.lwb,../../gemoc_studio/tests/org.eclipse.gemoc.studio.tests.system.mwb,../../gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.targetplatform\
										verify --errors --show-version  -Dstyle.color=always")					
							}
						}      
					}
				}
			}
			post {
				always {
					junit '**/target/surefire-reports/TEST-*.xml' 					
				}
				// archive artifact even if it failed (timeout) or was aborted (in order to debug using the video)
				// because the following steps will be skipped
				aborted {
				    archiveArtifacts 'gemoc-studio/dev_support/tycho_full_compilation/target/**, **/screenshots/**'
				}
				failure {
				    archiveArtifacts 'gemoc-studio/dev_support/tycho_full_compilation/target/**, **/screenshots/**'				    
				}
			}
	 	}
	}
	post {
		success {
			archiveArtifacts 'gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.updatesite/target/products/*.zip, gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.updatesite/target/repository/**, gemoc-studio/docs/org.eclipse.gemoc.studio.doc/target/publish/**, gemoc-studio/dev_support/tycho_full_compilation/target/**, **/screenshots/**, **/.metadata/.log, **/process_mem.log'
			deleteDir() /* clean up our workspace */            
        }
		fixed {
            slackSend channel: '#ci',
              color: 'good',
              message: "Build fixed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>) is back to normal."
        }
        unstable {
	        slackSend  channel: '#ci',
	          failOnError:true,
              color: 'warning',
	          message: "Build unstable  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
	    }
        failure {
	        slackSend  channel: '#ci',
	          failOnError:true,
              color: 'danger',
	          message: "Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
	    }
    }
}
