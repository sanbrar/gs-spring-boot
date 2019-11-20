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
        stage ('Artifactory configuration') {
            steps {
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",   //resolver-unique-id
                    serverId: "art-1",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )     
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",   //deployer-unique-id  
                    serverId: "art-1",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )
            } 
        }
        
        stage ('Feature Branch Version Check') {
            when {
                // Only say hello if a "greeting" is requested
                expression { env.BRANCH_NAME != 'master' }  //Not Master
            }
            steps {
                echo 'POM VERSION: ${POMVERSION}'                
                sh 'mvn versions:set versions:commit -DnewVersion="${POM_FILE_VERSION}-SNAPSHOT" -file="${POM_FILE_NAME}" '
            }
        }
                
        stage ('Maven build') {             //run the maven build, referencing the resolver and deployer we defined
            steps {
                rtMavenRun (
                   // tool: 'maven_tool', // Maven tool name from Jenkins configuration
                    pom: '${POM_FILE_NAME}',
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


