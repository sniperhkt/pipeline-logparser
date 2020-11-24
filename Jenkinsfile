// sample pipeline to test https://github.com/gdemengin/pipeline-logparser/ "Global Pipeline Library" logparser


// import logparser library
logparser = null

// ===============
// = constants   =
// ===============

timestamps {
    node("master") {
        stage("Setup") {
            try {
                deleteDir()
                checkout scm
                logparser = load "${pwd()}/logparser.groovy"
                logparser.setVerbose(true)
                echo "This is setup stage."
            } catch(e) {
                exceptionHandler(e,"Failed in Setup Stage")
            }
        }

        stage("Parallel Steps") {
            try {
                def parallelSteps = [:]
                for (def i = 0; i < 10; i++) {
                    def branchname = "branch-${i}"
                    parallelSteps[branchname] = {
                        node("master") {
                            try {
                                for (def j = 0; j < 10000; j++) {
                                    echo "In branch-${i}, count ${j}\n"
                                }
                            } catch(e) {
                                exceptionHandler(e, "Failed in branch-${i}")
                            }
                        }
                    }
                }
                parallel parallelSteps
            } catch(e) {
                exceptionHandler(e,"Failed in Parallel Steps")
            }
        }

        stage("Teardown") {
            try {
                for (def i = 0; i < 10; i++) {
                    def branchname = "branch-${i}"
                    logparser.archiveLogsWithBranchInfo("branch-${i}.log", [ filter: [branchname], hidePipeline: false ])
                }
                echo "This is teardown stage."
                archiveArtifacts artifacts: "/archive/*"
            } catch(e) {
                exceptionHandler(e,"Failed in Teardown Stage")
            }
        }
    }
}

String exceptionHandler(e, msg = "") {
    def w = new StringWriter()
    e.printStackTrace(new PrintWriter(w))
    currentBuild.result = 'UNSTABLE'
    seenException = true
    def eMsg = "${msg}\n${w.toString()}"
    println eMsg
    exceptions += "${eMsg}\n"
    return eMsg
}