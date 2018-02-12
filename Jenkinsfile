pipeline {
    agent any
    environment {
       def mvn_version = 'M3'
           def uploadSpec = """{
           "files": [
               {
               "pattern": "target/*.zip",
                   "target": "example-repo-local/"
               }
           ]
    }"""
        
        
    }
    stages {
        stage('Build') {
            steps { 
                configFileProvider(
                    [configFile(fileId: 'mulesoft-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
                    sh 'mvn -s $MAVEN_SETTINGS clean package -DskipTests=true'
                    }
                }
            }
        }
        stage('Upload Artifact') {
            steps {
                withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
                                 script {
                                        def server = Artifactory.newServer url: 'http://192.168.56.102:8081:8081/artifactory', credentialsId: 'mulesoft-artifactory'
                                        server.bypassProxy = true
                                        def buildInfo = server.upload spec: uploadSpec
                                        }
                    }
                }
            }
        stage('Deploy Test') {
            steps { 
                configFileProvider(
                    [configFile(fileId: 'mulesoft-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
                               withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'mule-builder', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                      sh 'mvn -s $MAVEN_SETTINGS -Ddeploy.username=$USERNAME -Ddeploy.password=$PASSWORD deploy -DskipTests=true'
                      }
                    }
                }
            }
        }
      }
}