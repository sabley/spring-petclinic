pipeline { 
    agent any
    tools {
      jdk 'jdk8'
    }
    stages {
        stage ('Build') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    ./mvnw spring-boot:run
                ''' 
            }
            post {
                always {
                    junit '**/target/surefire-reports/**/*.xml' 
                }      
            }
        }
        stage('Scan App - Build Container') {
            steps{
                parallel('IQ-BOM': {
                    echo '...'
                    /*nexusPolicyEvaluation failBuildOnNetworkError: false, 
                    iqApplication: 'petclinic', 
                    iqStage: 'build', 
                    iqScanPatterns: [[scanPattern: '']], 
                    jobCredentialsId: ''*/
                 },
                 'Static Analysis': {
                    echo '...run SonarQube or other SAST tools here'
                 },
                 'Build Container': {
                    echo '...need to learn the build process first'
                 })
            }

        }
        stage('Test Container') {
            steps{
                echo '...run container and test it'
            } 
            post {
                success {
                    echo '...the Test Scan Passed!'
                }
                failure {
                    echo '...the Test  FAILED'
                    error("...the Container Test FAILED")
                }
            }   
        }
        stage('Scan Container') {
            steps{
                echo '...TODO scan container'
                /* sh "docker save webgoat/webgoat-8.0 -o ${env.WORKSPACE}/webgoat.tar"

                nexusPolicyEvaluation failBuildOnNetworkError: false, 
                iqApplication: 'petclinic', 
                iqStage: 'stage', 
                iqScanPatterns: [[scanPattern: '*.tar']], 
                jobCredentialsId: '' */
            } 
            post {
                success {
                    echo '...the IQ Scan PASSED'
                    postGitHub commitId, 'success', 'analysis', 'Nexus Lifecycle Container Analysis succeeded',"${policyEvaluationResult.applicationCompositionReportUrl}"
                }
                failure {
                    echo '...the IQ Scan FAILED'
                    postGitHub commitId, 'failure', 'analysis', 'Nexus Lifecycle Containe Analysis failed',"${policyEvaluationResult.applicationCompositionReportUrl}"
                    error("...the IQ Scan FAILED")
                }
            } 
                /*  if (currentBuild.result == 'FAILURE'){
        postGitHub commitId, 'failure', 'analysis', 'Nexus Lifecycle Analysis failed',"${policyEvaluationResult.applicationCompositionReportUrl}"
        return
      } else {
        postGitHub commitId, 'success', 'analysis', 'Nexus Lifecycle Analysis succeeded',"${policyEvaluationResult.applicationCompositionReportUrl}"
      } */  
        }
        stage('Publish Container') {
            when {
                branch 'master'
            }
            steps {
                echo '...figure out container'
                /*sh '''
                    docker tag webgoat/webgoat-8.0 mycompany.com:5000/webgoat/webgoat-8.0:8.0
                    docker push mycompany.com:5000/webgoat/webgoat-8.0
                ''' */
            }
        }
    }
}
def postGitHub(commitId, state, context, description, target_url="http://localhost:8070") {
         def payload = JsonOutput.toJson(
           state: state,
           context: context,
           description: description,
           target_url: target_url
          )
        sh "curl -H \"Authorization: token ${gitHubApiToken}\" --request POST --data '${payload}' https://api.github.com/repos/${project}/statuses/${commitId}"
    /*  if (currentBuild.result == 'FAILURE'){
        postGitHub commitId, 'failure', 'analysis', 'Nexus Lifecycle Analysis failed',"${policyEvaluationResult.applicationCompositionReportUrl}"
        return
      } else {
        postGitHub commitId, 'success', 'analysis', 'Nexus Lifecycle Analysis succeeded',"${policyEvaluationResult.applicationCompositionReportUrl}"
      } */


        }
    
