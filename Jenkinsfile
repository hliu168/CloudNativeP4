pipeline {
    agent any
    
//     node{
//         echo "params.DEPLOY: ${params.DEPLOY} -- params.BRANCH_PORTAL_CLIENT: ${params.BRANCH_PORTAL_CLIENT} -- params.LAMBDAS: ${params.LAMBDAS}"
//     }
    
    
//     stages {
//         stage("Pre-Start") {
//             steps {
//                 script {
//                     echo "params.DEPLOY: ${params.DEPLOY} -- params.BRANCH_PORTAL_CLIENT: ${params.BRANCH_PORTAL_CLIENT} -- params.LAMBDAS: ${params.LAMBDAS}"
//                 }
//             }
//         }
//     }
    
//     script {
//         echo "params.DEPLOY: ${params.DEPLOY} -- params.BRANCH_PORTAL_CLIENT: ${params.BRANCH_PORTAL_CLIENT} -- params.LAMBDAS: ${params.LAMBDAS}"
//     }

//     echo "params.DEPLOY: ${params.DEPLOY} -- params.BRANCH_PORTAL_CLIENT: ${params.BRANCH_PORTAL_CLIENT} -- params.LAMBDAS: ${params.LAMBDAS}"

    
    options { 
        timestamps()
        ansiColor('xterm')
    }
    parameters {
//         script {
//             echo "params.DEPLOY: ${params.DEPLOY} -- params.BRANCH_PORTAL_CLIENT: ${params.BRANCH_PORTAL_CLIENT} -- params.LAMBDAS: ${params.LAMBDAS}"
//         }
        
//         echo "params.DEPLOY: ${params.DEPLOY} -- params.BRANCH_PORTAL_CLIENT: ${params.BRANCH_PORTAL_CLIENT} -- params.LAMBDAS: ${params.LAMBDAS}"

        booleanParam(name: 'DEPLOY', defaultValue: params.DEPLOY ?:false, description: '')
        string(name: 'BRANCH_PORTAL_CLIENT', defaultValue: params.BRANCH_PORTAL_CLIENT ?:'origin/integration', description: '')
        extendedChoice(defaultValue: params.LAMBDAS ?:'-ALL-', description: 'ListOfLambdas', multiSelectDelimiter: ',', name: 'LAMBDAS', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_MULTI_SELECT', value: '-ALL-,LambdaFolder1,LambdaFolder2,LambdaFolder3,LambdaFolder4,LambdaFolder5,LambdaFolder6,LambdaFolder7,LambdaFolder8,LambdaFolder9,LambdaFolder10,LambdaFolder11,LambdaFolder12,LambdaFolder13,LambdaFolder14,LambdaFolder15,LambdaFolder16,LambdaFolder17,LambdaFolder18,LambdaFolder19,LambdaFolder20', visibleItemCount: 20)
    }
    stages {
        stage("Prepare Workspace") {
            steps {
                script {
                    cleanWs()
                    deleteDir()
                    checkout scm
                    def APP_BASE_VER = env.APP_BASE_VER
                    def buildNumber = "${APP_BASE_VER}.${BUILD_NUMBER}"
                    currentBuild.displayName = "TestPipeline (${buildNumber})"
                  
                    def repoInfo = [
                    //    "/"		:	[ 'url': 'https://github.com/hliu168/CloudNativeP4.git', 'branch': "master" ],
                        "web"   :   [ 'url': 'https://github.com/hliu168/web.git',  'branch': "master"] 
                    ]
                    
                    def stepsForParallel = [:]
                    repoInfo.each { repoName, repoUrl ->
                        stepsForParallel[repoName] = { ->
                            dir("${repoName}") {
                                checkout([$class: 'GitSCM',
                                          branches: [[name: repoUrl['branch'] ]],
                                          //extensions: [[$class: 'LocalBranch', localBranch: "**"]],
                                          userRemoteConfigs: [[ /*credentialsId: "${credScm}", */url: repoUrl['url']]]])
                            }
                        }
                        echo "${repoUrl} checked out to ${WORKSPACE}/${repoName}/ "
                    }
                    // Run all checkoutsin parallel
                    parallel stepsForParallel
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
