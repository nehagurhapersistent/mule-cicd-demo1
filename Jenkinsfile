pipeline {
    agent any
    tools {
        maven "maven"
    }
    stages {
         stage('Build') {
            steps {
               bat "mvn -B -U -e -V clean -DskipTests package"
            }
        }
        stage('Test') {
            steps {
             
               bat "mvn test -Dhttp.port=8093"
            }
        }
        stage('Publish to Nexus Repository') {
			
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.jar");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
			propertyFileByGlob = findFiles(glob: "*.properties");
			propertiesFile =  propertyFileByGlob[0].path;
			
			postmanCollectionByGlob = findFiles(glob: "mule-jenkins-tests.postman_collection.json");
			postmanCollectionFile =  postmanCollectionByGlob[0].path;
			echo "${postmanCollectionFile}"
			
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
			    version: "${BUILD_NUMBER}",
                            repository: NEXUS_REPOSITORY,
                            credentialsId: "NEXUS_CREDENTIALS",
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: "jar"],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: propertiesFile,
                                type: "properties"]
								
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
		}	
	}
	post {  
         success {  
             mail bcc: '', body: "<b>Build Success</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "Build Success: Project name -> ${env.JOB_NAME}", to: "enggnehagurha88@gmail.com";
         }  
         failure {  
             mail bcc: '', body: "<b>Build Failure</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "Build Failure: Project name -> ${env.JOB_NAME}", to: "enggnehagurha88@gmail.com";  
         }  
     }  
}

