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
   }
}