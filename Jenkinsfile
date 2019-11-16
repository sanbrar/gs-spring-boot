pipeline {
    agent any
    stages {
        stage ('Clone') {
            steps {
                git branch: 'master', url: "https://github.com/sanbrar/gs-spring-boot.git"
            }
        }

        stage ('Artifactory configuration') {
        
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "art-1",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "art-1",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )

        }

        stage ('Exec Maven') {
            steps {
                rtMavenRun (
                    tool: maven_tool, // Maven tool name from Jenkins configuration
                    pom: 'complete/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
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
