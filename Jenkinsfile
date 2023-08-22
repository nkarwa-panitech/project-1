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

      maven '3.9.4'
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
       stage('Quality Gate Statuc Check'){
                  steps{
                      script{
                      withSonarQubeEnv('sonarqube-8.9.2') { 
                      sh "mvn sonar:sonar"
                       }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
		    sh "mvn clean install"
                  }
                }  
              }
       stage("Build"){
// 	agent {
//         docker { image 'maven:3.8.1-adoptopenjdk-11' }
//          }

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

            sshPublisher(publishers: [sshPublisherDesc(configName: 'dev-tomcat', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
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
		          withCredentials([string(credentialsId: 'dockerhublogin', variable: 'docker_password')]) {		    
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

   post{
        always {
            slackSend( color: "good", message: "I have done my job")
        }
        success{
            slackSend( color: "good", message: "${custom_msg()}")
        }
        failure{
            slackSend( color: "danger", message: "${custom_msg()}")
        }


}

}
def custom_msg()
{
  def JENKINS_URL= "http://3.233.241.149:8080/"
  def JOB_NAME = env.JOB_NAME
  def BUILD_ID= env.BUILD_ID
  def JENKINS_LOG= " Success: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/consoleText"
  return JENKINS_LOG
}
