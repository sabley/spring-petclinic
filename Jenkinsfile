pipeline {
  agent any
  stages {
   stage('Build') {
     steps {
       sh '''
                   echo "PATH = ${PATH}"
                   echo "M2_HOME = ${M2_HOME}"
                   mvn help:evaluate -Dexpression=settings.localRepository
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
      parallel {
        stage ('Publish to Build Tag 125') {
          steps {
              nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '2.0.0']]], tagName: 'build-125'
          }     
        }
        stage ('Publish to Build Tag 123') {
          steps {
           nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '1.0.0']]], tagName: 'build-123'
          }
        }
        stage ('Publish to Build Tag 120') {
          steps {
           nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '0.0.1']]], tagName: 'build-120'
          }
        } 
      }
    }
    stage ('Next Steps') {
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
            sh ''' 
               docker build -t test-boot .
            '''
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
    stage('Clean Up Validation') {
      steps {
        input "Clean Up??"
      }
    }
    stage('Delete all the things') {
      steps {
        deleteComponents nexusInstanceId: 'nx3', tagName: 'build-125'
        deleteComponents nexusInstanceId: 'nx3', tagName: 'build-123'
        sh '''
            curl --verbose -u admin:admin123 -X DELETE "http://nexus:8081/service/rest/v1/tags/build-125" -H "accept: application/json"
            curl --verbose -u admin:admin123 -X DELETE "http://nexus:8081/service/rest/v1/tags/build-123" -H "accept: application/json"
        '''
      }
    }
  }
  tools {
    maven 'M3'
  }
}