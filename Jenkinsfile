pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    environment {
        MAVEN_HOME = '/usr/share/maven'
        GLOBAL_RELEASE_REPO = 'libs-release'
        GLOBAL_SNAPSHOT_REPO = 'libs-snapshot'
        PRJ_RELEASE_REPO = 'libs-release-local'
        PRJ_SNAPSHOT_REPO = 'libs-snapshot-local'
        POM_FILE_NAME = 'complete/pom.xml'
        POM_FILE_VERSION = sh(script: 'mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -file="complete/pom.xml" -Dexpression=project.version | grep -e "^[^[]" ', returnStdout: true).trim()
        ARTIFACTORY_PATH = 'libs-snapshot-local/lll/springframework/gs-spring-boot'
        GET_ON_GIT_COMMIT_NUM = 'fe192efa59b6004f24ec090fb401871784b31bd8'
    }
    stages {
        stage ('Artifactory configuration') {
            steps {
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",   //resolver-unique-id
                    serverId: "art-1",
                    releaseRepo: "${GLOBAL_RELEASE_REPO}",
                    snapshotRepo: "${GLOBAL_SNAPSHOT_REPO}"
                )     
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",   //deployer-unique-id  
                    serverId: "art-1",
                    releaseRepo: "${PRJ_RELEASE_REPO}",
                    snapshotRepo: "${PRJ_SNAPSHOT_REPO}"
                )
                //To set the Build-Info object to automatically capture environment variables while downloading and uploading files
                rtBuildInfo (
                    captureEnv: true
                )                
                
                
            } 
        }
        
        stage ('Feature Branch Version Check') {
            environment {
                POM_VERSION_SNAPSHOT = sh(script: 'echo $(echo "${POM_FILE_VERSION}" | cut -d"-" -f1)-${GIT_COMMIT}-SNAPSHOT', returnStdout: true).trim()
            }
            when {
                // Only run if branch is not a master
                expression { env.BRANCH_NAME != 'master' } 
            }
            steps {
                // Add the word "SNAPSHOT" to the POM version
                sh 'mvn versions:set versions:commit -DnewVersion="${POM_VERSION_SNAPSHOT}" -file="${POM_FILE_NAME}" '
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
        
        stage ('Test getting a file with git commit number') {
            steps {
            rtDownload (
                serverId: "art-1",
                spec: """{
                      "files": [
                        {
                          "pattern": "${ARTIFACTORY_PATH}/*${GET_ON_GIT_COMMIT_NUM}*",
                          "target": "bazinga/",
                          "props": "type=pom"
                        }
                      ]
                }""" ,
                failNoOp: true
            )
            }
        }
    }
}


