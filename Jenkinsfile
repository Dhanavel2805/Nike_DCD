pipeline {
    agent any

    stages {
        stage('Pull Code from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Dhanavel2805/Nike_DCD.git'
            }
        }

        stage('Verify JMeter Scripts') {
            steps {
                bat 'dir /s /b *.jmx'
            }
        }

        stage('Execute Selected JMeter Scripts in Parallel') {
            steps {
                script {
                    def selectedScripts = params.JMETER_SCRIPTS

                    if (!selectedScripts?.trim()) {
                        error 'No JMeter scripts selected. Check the parameter name and selected values.'
                    }

                    def scripts = selectedScripts.split(',')
                        .collect { it.trim() }
                        .findAll { it }

                    def jmeter = 'C:/Users/LENOVO/OneDrive - Radiant Global Technology/Project/apache-jmeter-5.6.3_/apache-jmeter-5.6.3/bin/jmeter.bat'

                    def buildFolder = "results/Build_${env.BUILD_NUMBER}"
                    def parallelJobs = [:]

                    scripts.each { selected ->
                        def scriptPath = selected
                        def scriptName = scriptPath.tokenize('/\\').last()
                        def baseName = scriptName.replaceAll(/(?i)\.jmx$/, '')
                        def safeName = baseName.replaceAll(/[^A-Za-z0-9_.-]/, '_')

                        parallelJobs[safeName] = {
                            stage("Run ${safeName}") {
                                def scriptFile = scriptPath.replace('/', '\\')
                                def resultFile = "${buildFolder}/${safeName}.jtl"

                                if (!fileExists(scriptFile)) {
                                    error "JMeter script not found: ${scriptFile}"
                                }

                                bat """
                                    if not exist "${buildFolder}" mkdir "${buildFolder}"

                                    echo Running JMeter script: ${scriptFile}
                                    echo JTL output: ${resultFile}

                                    call "${jmeter}" -n -t "${scriptFile}" -l "${resultFile}"
                                    set "JMETER_EXIT_CODE=%ERRORLEVEL%"

                                    echo Checking generated JTL file...

                                    if exist "${resultFile}" (
                                        echo SUCCESS: JTL file exists.
                                        dir "${buildFolder}"
                                    ) else (
                                        echo ERROR: JTL file was not found.
                                        echo Current directory:
                                        cd
                                        echo Searching workspace for JTL files...
                                        dir /s /b *.jtl
                                        exit /b 1
                                    )

                                    if not "%JMETER_EXIT_CODE%"=="0" (
                                        echo ERROR: JMeter exited with code %JMETER_EXIT_CODE%.
                                        exit /b %JMETER_EXIT_CODE%
                                    )
                                """
                            }
                        }
                    }

                    parallel parallelJobs
                }
            }
        }
    }

    post {
        always {
            echo 'Archiving JMeter JTL files...'

            archiveArtifacts(
                artifacts: "results/Build_${env.BUILD_NUMBER}/**/*.jtl",
                allowEmptyArchive: true,
                fingerprint: true
            )

            echo 'Check Build Artifacts for the current build JTL files.'
        }
    }
}
