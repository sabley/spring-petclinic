pipeline {
  agent any
  stages {
   stage('Build') {
     steps {
       sh '''
                   echo "PATH = ${PATH}"
                   echo "M2_HOME = ${M2_HOME}"
                   mvn versions:set -DnewVersion=2.0.0
                   mvn package -B -DskipTests=true
               '''
       }
     }
      stage ('Creating build tag') {
        steps {
            createTag nexusInstanceId: 'nx3', tagAttributesJson: '{"createdBy" : "Moose"}', tagName: 'build-123'
            createTag nexusInstanceId: 'nx3', tagAttributesJson: '{"createdBy" : "Moose"}', tagName: 'build-125'
        }
    }
    stage ('Publishing') {
      parallel (
        steps {
            nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '2.0.0']]], tagName: 'build-125'
            }
            steps {
             nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '1.0.0']]], tagName: 'build-123'
             }
            steps {
             nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '0.0.1']]], tagName: 'build-120'
            }
          )
        }
    
    stage ('Move') {
        steps {
            input "Deploy to Prod?"
            moveComponents destination: 'maven-test', nexusInstanceId: 'nx3', tagName: 'build-123'
        }
    }
    stage ('Delete') {
        steps {
            deleteComponents nexusInstanceId: 'nx3', tagName: 'build-120'
        }
    } 

    stage('Scan App - Build Container') {
      parallel {
        stage('IQ-BOM') {
          steps {
            nexusPolicyEvaluation(iqApplication: 'petclinic', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
        }
        stage('Static Analysis') {
          steps {
            echo '...run SonarQube or other SAST tools here'
          }
        }
        stage('Build Container') {
          steps {
            echo '...need to learn the build process first'
          }
        }
      }
    }
    stage('Test Container') {
      steps {
        echo '...run container and test it'
      }
      post {
        success {
          echo '...the Test Scan Passed!'

        }

        failure {
          echo '...the Test  FAILED'
          error '...the Container Test FAILED'

        }

      }
    }
    stage('Scan Container') {
      steps {
        echo '...TODO scan container'
      }
      post {
        success {
          echo '...the IQ Scan PASSED'
          postGitHub(commitId, 'success', 'analysis', 'Nexus Lifecycle Container Analysis succeeded', "${policyEvaluationResult.applicationCompositionReportUrl}")

        }

        failure {
          echo '...the IQ Scan FAILED'
          postGitHub(commitId, 'failure', 'analysis', 'Nexus Lifecycle Containe Analysis failed', "${policyEvaluationResult.applicationCompositionReportUrl}")
          error '...the IQ Scan FAILED'

        }

      }
    }
    stage('Publish Container') {
      when {
        branch 'master'
      }
      steps {
        echo '...figure out container'
      }
    }
  }
  tools {
    maven 'M3'
  }
}