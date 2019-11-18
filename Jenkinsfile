pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    environment {
        MAVEN_HOME = '/usr/share/maven'
    }
    stages {
       // stage ('Clone') {
         //   steps {
                //git branch: 'master', url: "https://github.com/sanbrar/gs-spring-boot.git"  //${env.BRANCH_NAME}
                //Hello
           // }
       // }

        stage ('Speak') {
            when {
                // Only say hello if a "greeting" is requested
                expression { env.BRANCH_NAME == 'master' }
            }
            steps {
                echo "master branch"
            }
        }
        
        
        stage ('Artifactory configuration') {
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",   //deployer-unique-id  -- edit master TEST2
                    serverId: "art-1",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",   //resolver-unique-id
                    serverId: "art-1",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
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
