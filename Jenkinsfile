pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    
    
     environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.1.17:8081"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    }
    
     stages {
        stage('Git Checkout') {
            steps {
                git branch : 'operateur' ,
                    url : 'https://github.com/Muhammed-Alaa-Kanzari/achat-back';
            }
        }
        
         stage('Cleaning the project') {
            steps{
                sh 'mvn clean'
            }
        }
        
        stage (	'Unit testing ' ) {
			   steps{
				 sh 'mvn test'  		 
				}
			}	
			
			
        stage ('Maven Build'){	
		  	steps{
		  		  sh 'mvn clean install '
		  	 }
		 }
		 
		  stage ('Artifact construction') {
            steps{
                sh 'mvn  package'
            }
        }
		 
		 stage ('SonarQube analysis'){	
		  	steps{
		  		script{
		  				withSonarQubeEnv(credentialsId: 'sonarToken', installationName: 'sonarServer') {		  			
		  		       	sh 'mvn clean package sonar:sonar '
		            		}
		              }		
		  	      }	
		   }
		   
		   	
		   
		   stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
		   
		  
		  
		    stage ('Build our image'){
            steps{
                sh 'sudo docker build --build-arg IP=0.0.0.0 -t medalaakanzari/achat_back .'
            }
        }
        
        
        stage ('Deploy our image'){
            steps{
            
            
                sh 'sudo docker login -u samiriahi -p 123456799';
                
                sh 'sudo docker push samiriahi/achat_back'
                
                }
            }
    }
		  
		   	
		   
		   	
    }
         
}



