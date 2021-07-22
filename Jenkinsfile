pipeline {
    agent any
    options { 
        timestamps()
        ansiColor('xterm')
    }
    parameters {
        booleanParam(name: 'DEPLOY', defaultValue: true, description: '')
        extendedChoice(defaultValue: 'Lambda1,Lambda2', description: 'ListOfLambdas', multiSelectDelimiter: ',', name: 'LAMBDAS', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_MULTI_SELECT', value: 'Lambda1,  Lambda2, Lambda3,Lambda4,Lambda5,Lambda6,Lambda7,Lambda8,Lambda9,Lambda10, Lambda11', visibleItemCount: 20)
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
                    
                    sh '''#!/bin/bash
                    set -eu
                    command_not_existing
                    '''
                    
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
            build job: 'DownStreamJob', parameters: [
                string(name: 'BRANCH_PORTAL_CLIENT', value: 'origin/integration'),
                string(name: 'AGENT_VERSION', value: '4.4.1932'),
                string(name: 'PORTAL_ENV', value: 'integ')
            ], wait: false
        }
        failure {
            echo "++++++++++++++++++++++++Failure in Build++++++++++++++++++++++++"
        }
    }
}
