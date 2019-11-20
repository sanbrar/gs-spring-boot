//def pomFile = "pom.xml"
//def POM_FILE_VERSION = "2.0.0"

pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    environment {
        MAVEN_HOME = '/usr/share/maven'
        POM_FILE_NAME = 'complete/pom.xml'
        POM_FILE_VERSION = sh(script: 'mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -file="complete/pom.xml" -Dexpression=project.version | grep -e "^[^[]" ', returnStdout: true).trim()
    }
    stages {
       
        stage('Set Variables') {
          steps {
              
           // script {
         //       echo "pomFile Name before setting ${pomFile}"                  
         //   }
           
           // script {
           //     pomFile = "complete/pom.xml"
          //  }
                            
            script {
                echo "After setting the pomFile: ${POM_FILE_NAME}"
            } 
              
            script {
                echo "pomVersion before using function to set pomVersion ${POM_FILE_VERSION}"
            } 
              
            //script {
            //    POM_FILE_VERSION = sh 'mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate  -file="${POM_FILE_NAME}" -Dexpression=project.version | grep -e "^[^[]" '
            //} 
              
            script {
                echo "After setting the pomVersion: ${POM_FILE_VERSION}"
            }
          }
        }        
        
        stage ('Artifactory configuration') {
            steps {
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",   //resolver-unique-id
                    serverId: "art-1",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )                
            } 
        }
        
        stage ('Artifactory Config - Master') {
            when {
                // Only say hello if a "greeting" is requested
                expression { env.BRANCH_NAME == 'master' }
            }
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",   //deployer-unique-id  
                    serverId: "art-1",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )
            }            
        }
        
        stage ('Artifactory Config - Feature') {
            when {
                // Only say hello if a "greeting" is requested
                expression { env.BRANCH_NAME != 'master' }  //Not Master
            }
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",   //deployer-unique-id  -- edit master TEST2
                    serverId: "art-1",
                    releaseRepo: "libs-snapshot-local",
                    snapshotRepo: "libs-snapshot-local"
                )
            }
        }
        
        
        stage ('Verify Build Version Number') {
            steps { 
                echo 'POM VERSION: ${POMVERSION}'                
                sh 'mvn versions:set versions:commit -DnewVersion="${POM_FILE_VERSION}-SNAPSHOT" -file="complete/pom.xml"'

               // sh 'mvn scm:checkin -Dincludes=complete/pom.xml -Dmessage="Setting version, preping for release."'
                
            }
        }
                
        stage ('Maven build') {             //run the maven build, referencing the resolver and deployer we defined
            steps {
                rtMavenRun (
                   // tool: 'maven_tool', // Maven tool name from Jenkins configuration
                    pom: 'complete/pom.xml',
                    goals: 'clean install',
                    // Maven options.
                    opts: '-Xms1024m -Xmx4096m',
                    deployerId: 'MAVEN_DEPLOYER',
                    resolverId: 'MAVEN_RESOLVER'
                )
            }
        } //jjj

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "art-1"
                )
            }
        }    
    }
}


