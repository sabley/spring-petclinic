pipeline {
  agent any
  stages {
   stage('Build a Snapshot') {
     steps {
       sh '''
                   echo "PATH = ${PATH}"
                   echo "M2_HOME = ${M2_HOME}"
                   mvn help:evaluate -Dexpression=settings.localRepository
                   mvn package -B -DskipTests=true
               '''
       }
     }
  stage('Build a Release') {
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
  stage ('Upload to Nexus Repository') {
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
  stage ('Move Release to Test') {
    steps {
        input "Deploy to Prod?"
        moveComponents destination: 'maven-test', nexusInstanceId: 'nx3', tagName: 'build-123'
    }
  }
  stage ('Delete Bad Version') {
    steps {
          deleteComponents nexusInstanceId: 'nx3', tagName: 'build-120'
    }
  } 
  stage('LifeCycleScan') {
      stage('Lifecycle Scan') {
        steps {
          nexusPolicyEvaluation(iqApplication: 'petclinic', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
        }
      }
  }
  stage('Clean Up Checkpoint') {
    steps {
      input "Clean Up??"
    }
  }
  stage('Delete all the things') {
    steps {
      deleteComponents nexusInstanceId: 'nx3', tagName: 'build-125'
      deleteComponents nexusInstanceId: 'nx3', tagName: 'build-123'
      sh '''
          curl --verbose -u admin:admin123 -X DELETE "http://ec2-54-165-203-133.compute-1.amazonaws.com:8081/service/rest/v1/tags/build-125" -H "accept: application/json"
          curl --verbose -u admin:admin123 -X DELETE "http://ec2-54-165-203-133.compute-1.amazonaws.com:8081/service/rest/v1/tags/build-123" -H "accept: application/json"
      '''
    }
  }
  }
  tools {
    maven 'M3'
  }
}