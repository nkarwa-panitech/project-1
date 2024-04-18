currentBuild.displayName = "Build # "+currentBuild.number

   def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }
pipeline{

   agent { label 'node01' }
   environment{
	    Docker_tag = getDockerTag()
        }
   tools{

      maven '3.9.6'
   }

   stages{
       stage("Checkout"){
        steps{
             git url: 'https://github.com/nkarwa-panitech/project-1.git', branch: 'main'
             sh "ls -ll"
        }
       }
       stage("UnitTest"){
	    agent {
          docker { image 'maven:3.8.1-adoptopenjdk-11' }
        }
        steps{
            sh "mvn test"
        }
       }
       stage('Quality Gate Status Check'){
                  steps{
                      script{
                      withSonarQubeEnv('sonar') { 
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
       stage("Build"){
	    // agent {
        //     docker { image 'maven:3.8.1-adoptopenjdk-11' }
        //      }
        steps{
            sh 'mvn package'
        }
       }
       stage("ArchiveArtifacts"){
        steps{
               archiveArtifacts artifacts: 'target/WebApp.war', followSymlinks: false
        }
       }
       stage("NexusPublisher"){

        steps{

            echo "Uploading artifacts to Nexus"
        }
       }

       stage("Deploy Dev Env"){

        steps{

            sshPublisher(publishers: [sshPublisherDesc(configName: 'tomcat', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        }
       
       }
       stage("Approval"){

        steps{

            timeout(time: 15, unit: 'MINUTES'){ 
	        input message: 'Do you approve deployment for production?' , ok: 'Yes'}

        }
       }

       stage("Prod build"){
        steps{
            script{
                   sh 'docker build . -t nkarwapanitech/devops-training:$Docker_tag'
		          withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'DOCKER_REGISTRY_PWD', usernameVariable: 'DOCKER_REGISTRY_USER')]) {		    
				  sh 'docker login -u nkarwapanitech -p $docker_password'
				  sh 'docker push nkarwapanitech/devops-training:$Docker_tag'
			        }
                    }
                }
        }

       stage('ansible playbook'){
	        agent { label 'node01' }
			steps{
			 	script{
				    sh '''final_tag=$(echo $Docker_tag | tr -d ' ')
				     echo ${final_tag}test
				     sed -i "s/docker_tag/$final_tag/g"  deployment.yaml
				     '''
				    ansiblePlaybook become: true, installation: 'ansible', inventory: 'hosts', playbook: 'ansible.yaml'
				}
			}

   }
   }
}