pipeline {
    agent any
    environment {
        ENV_DEPLOY = sh(script:"echo -n ${params.DEPLOY}", returnStdout: true)
        ENV_BRANCH = sh(script:"echo -n ${params.BRANCH_PORTAL_CLIENT}", returnStdout: true)
        ENV_LAMBDAS = sh(script:"echo -n ${params.LAMBDAS}", returnStdout: true)
    }
    options { 
        timestamps()
        ansiColor('xterm')
    }
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'origin/integration', description: '')
        booleanParam(name: 'DEPLOY', defaultValue: params.DEPLOY ?:false, description: '')
        string(name: 'BRANCH_PORTAL_CLIENT', defaultValue: params.BRANCH_PORTAL_CLIENT ?:'origin/integration', description: '')
        extendedChoice(defaultValue: params.LAMBDAS ?:'-ALL-', description: 'ListOfLambdas', multiSelectDelimiter: ',', name: 'LAMBDAS', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_MULTI_SELECT', value: '-ALL-,LambdaFolder1,LambdaFolder2,LambdaFolder3,LambdaFolder4,LambdaFolder5,LambdaFolder6,LambdaFolder7,LambdaFolder8,LambdaFolder9,LambdaFolder10,LambdaFolder11,LambdaFolder12,LambdaFolder13,LambdaFolder14,LambdaFolder15,LambdaFolder16,LambdaFolder17,LambdaFolder18,LambdaFolder19,LambdaFolder20', visibleItemCount: 20)
    }
    stages {
        stage("Prepare Workspace") {
            steps {
                script {
                    echo "Hello Pipeline"
                    echo "DEPLOY: ${ENV_DEPLOY} -- BRANCH_PORTAL_CLIENT: ${ENV_BRANCH} -- LAMBDAS: ${ENV_LAMBDAS}"
                    cleanWs()
                    deleteDir()
                    checkout scm
                    def APP_BASE_VER = env.APP_BASE_VER
                    def buildNumber = "${APP_BASE_VER}.${BUILD_NUMBER}"
                    currentBuild.displayName = "TestPipeline (${buildNumber})"
                  
                }
            }
        }
        stage("Build") {
            steps {
                // script {
                //     try {
                    nodejs(nodeJSInstallationName: "16.5.0") {

                        sh '''#!/bin/bash
                        set -eu
                        
                        npm -v
                        IFS=',' read -r -a ITEMS <<< $LAMBDAS
                        for item in ${ITEMS[@]}
                        do
                            if [ $item == "-ALL-" ]
                            then
                                LAMBDAS='LambdaFolder1,LambdaFolder2,LambdaFolder3,LambdaFolder4,LambdaFolder5,LambdaFolder6,LambdaFolder7,LambdaFolder8,LambdaFolder9,LambdaFolder10,LambdaFolder11,LambdaFolder12,LambdaFolder13,LambdaFolder14,LambdaFolder15,LambdaFolder16,LambdaFolder17,LambdaFolder18,LambdaFolder19,LambdaFolder20'
                                break
                            fi
                        done
                        IFS=',' read -r -a LAMB_ARRAY <<< $LAMBDAS
                        # A_Command_supposed_to_fail
                        echo $LAMBDAS
                        for lambda in ${LAMB_ARRAY[@]}
                        do
                            # A_Command_supposed_to_fail
                            echo $lambda
                        done
                        '''
                        sh "echo 'SUCCESS' > build_status"
                    }
                
                script {
                    def ret = sh(script: "cat package.json | jq -r '.name'", returnStdout: true)
                    echo "${ret}"
                }
                    // }
                    // catch (Exception e) {
                    //     sh "echo 'FAILURE' > build_status"
                    //     error('Build Failed' + e.toString())
                    // }
                    // finally {
                    //     def data = readFile(file: 'build_status')
                    //     echo "In finally, build status is ${data}!"
                    // }
                // }
            }
        }
        stage ("Deploying") {
             when {
                allOf {
          //          branch 'master'
                    expression { params.DEPLOY == true }
                }
            }
            steps {
                echo 'Deploying'
            }
        } 
    }
    post {
        success {
            script {
//                 def lambdas = "${LAMBDAS}".split(",")
//                 def artifactList = []
//                 lambdas.each {l->
//                     echo "$l"
//                     def artifactName = sh(script: "cat $l/package.json | jq -r '.name'", returnStdout: true)
//                     echo "$artifactName"
//                     artifactList.add(artifactName.trim())
//                 }
                
//                 def artifactString = artifactList.join(",")
//                 echo "$artifactString"

                def APP_BASE_VER = env.APP_BASE_VER
                build job: 'DownStreamJob', parameters: [
                    string(name: 'BRANCH_PORTAL_CLIENT', value: params.BRANCH_PORTAL_CLIENT),
                    string(name: 'AGENT_VERSION', value: "${APP_BASE_VER}.${BUILD_NUMBER}"),
                    string(name: 'PORTAL_ENV', value: 'integ'),
                    string(name: 'ARTIFACT_LIST', value: "$LAMBDAS")
                ], wait: false
            }
        }
        failure {
            echo "++++++++++++++++++++++++Failure in Build++++++++++++++++++++++++"
        }
    }
}
