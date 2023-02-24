currentBuild.displayName = "Java_Demo_Webapp # "+currentBuild.number

   def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }
        

pipeline{
        agent {
		docker {
          		image 'maven'
          		args '-v $HOME/.m2:/root/.m2'
          	}
            } 
        environment{
	    Docker_tag = getDockerTag()
        }

// # SonaQube Quality Gate:        
        stages{
              stage('SonaQube Quality Gate Statuc Check'){
                  steps{
                      script{
                      withSonarQubeEnv('sonarserver') { 
                      sh "mvn sonar:sonar"
                       }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                  }
                }  
              }

// # Build Maven Jar files
//  #-----------  knote-java   ----------------        
              stage('Build knote-java Maven App'){
              steps{
                  script{
		   sh "mvn clean install"
                       }
                    }
                 }


// # Build Docker Images
// 1#--------------- knote-java:
              stage('Build knote-java Image'){
              steps{
                  script{
		   sh 'docker build . -t devtraining/knote-java:${BUILD_NUMBER}'
                       }
                    }
                 }
// #--------------- knote-java image test			 
			//   stage('Test image') {
			// 	clientImage.inside {
			// 	  sh 'echo "Tests passed"'
			// 	}
			//   }
		 
                 
// # Tags Push Docker Images	
	 
//  1#--------------- knote-java:		
              stage('Push knote-java Image'){
              steps{
                  script{
		   docker.withRegistry("https://index.docker.io/v1/", "Docker_Hub" ) {
                   sh 'docker push devtraining/knote-java:${BUILD_NUMBER}'
			}
                       }
                    }
                 }


// # Knote Java Deploy Job


// #-----------------
		stage('Build knote-java Deploy Job'){
		steps{
		    script{
		      echo "triggering Knote Deploy Build Job"
		    build job: 'knote-deploy-build', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
		    }
		}
	     }
          }
}