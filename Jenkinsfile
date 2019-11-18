def getPomVersion(String pomFile = 'pom.xml') {

    def version = sh 'mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -Dexpression=project.version -file="${pomFile}" | grep -e "^[^[]" '

        // pom_version = sh 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout -file="complete/pom.xml"', returnStdout: true
        
        // POMVERSION = sh 'mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -Dexpression=project.version -file="complete/pom.xml" | grep -e "^[^[]" '
        
        
        //pom_version = sh mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -Dexpression=project.version -file="complete/pom.xml" | grep -v '\['
        
        //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
        //POM_IMAGE = readMavenPom().getArtifactId()
        //POM_VERSION = readMavenPom().getVersion()   

    return '2.0.0'

}

pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    environment {
        MAVEN_HOME = '/usr/share/maven'

        POMVERSION = getPomVersion('complete/pom.xml')

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
                sh 'mvn versions:set versions:commit -DnewVersion="${POMVERSION}-SNAPSHOT" -file="complete/pom.xml"'

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
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "art-1"
                )
            }
        }    
    }
}





