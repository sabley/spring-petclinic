pipeline {
  agent any
  stages {
    stage('Build a Release') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    mvn versions:set -DnewVersion=2.0.0
                    mvn package -B -DskipTests=true
                '''
        }
    }
    stage('LifeCycle Scan - CI') {
          steps {
            nexusPolicyEvaluation(iqApplication: 'petclinic', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
    }
    stage ('Creating build tag') {
          steps {
              createTag nexusInstanceId: 'nx3', tagAttributesJson: '{ "name": "project-abc-142", "attributes": { "jvm": "9", "built-by": "jenkins", "tech-owner": "Moose", "Jira Tickets": { "Ticket-1234": "Link", "Ticket-1235": "Link", "Ticket-1236": "Link"}}}', tagName: 'build-123'
              createTag nexusInstanceId: 'nx3', tagAttributesJson: '{ "name": "project-abc-142", "attributes": { "jvm": "9", "built-by": "jenkins", "tech-owner": "Moose"}}', tagName: 'build-125'
          }
    }
    stage ('Upload to Nexus Repository') {
      parallel {
        stage ('Publish to Build Tag 125') {
          steps {
              nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-prod', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '2.0.0']]], tagName: 'build-125'
          }     
        }
        stage ('Publish to Build Tag 123') {
          steps {
            nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-dev', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '1.0.0']]], tagName: 'build-123'
          }
        }
        stage ('Publish to Build Tag 120') {
          steps {
            nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-dev', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/spring-petclinic-2.0.0.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '0.0.1']]], tagName: 'build-120'
          }
        } 
      }
    }
    stage ('Move Release to Test') {
      steps {
          input "Deploy to Prod?"
          moveComponents destination: 'maven-qa', nexusInstanceId: 'nx3', tagName: 'build-123'
      }
    }
    stage ('Delete Bad Version') {
      steps {
            deleteComponents nexusInstanceId: 'nx3', tagName: 'build-120'
      }
    }
    stage('Lifecycle Scan Release') {
      steps {
        nexusPolicyEvaluation(iqApplication: 'petclinic', iqStage: 'release', iqScanPatterns: [[scanPattern: '']])
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