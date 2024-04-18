pipeline{

   agent { label 'node01' }
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

   }
}