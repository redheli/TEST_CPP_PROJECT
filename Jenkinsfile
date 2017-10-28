pipeline {

agent any
stages {
    stage('Build All') {
        steps {
            echo "Building ${env.BUILD_DIR}"
            sh """#!/usr/bin/env bash
                set -eux
                mkdir build
                cd build
                cmake ..
                make
                popd
            """
        }
    }
    stage('Build Tests') {
        steps {
            echo "Building Tests ${env.BUILD_DIR}"
            sh """#!/usr/bin/env bash
                source /nuenv.sh
                set -eux
                pushd nucore/
                make -j8 USE_CUDA=ON WITH_PERC=ON WITH_NUMAP=ON CCACHE_DISABLE=ON test-build-only
                popd
            """
        }
    }
    stage('Run Tests'){
        steps {
            parallel(
                unitTests: {
                    timeout(time: 60, unit: 'MINUTES') {
                        sh """#!/usr/bin/env bash
                            source /nuenv.sh
                            set -eux
                            pushd nucore/pod-build
                            ctest --output-on-failure
                            popd
                        """
                    }
                },
                pythonTests: {
                    sh """#!/usr/bin/env bash
                        source /nuenv.sh
                        set -eux
                        pushd nucore
                        nosetests -v tests/unit/python
                        popd
                    """
                },
                staticAnalysis: {
                    sh """#!/usr/bin/env bash
                        source /nuenv.sh
                        set -eux
                        pushd nucore
                        ./scripts/static-analysis/cppcheck.sh
                        # ./scripts/static-analysis/clang-scan-build.sh  # clang is failing for some reason
                        cloc --by-file --xml --exclude-dir=src/3rdparty --out=static-analysis/cloc.xml \${NUCORE}/nucore/src
                        popd
                    """
                }
            )
        }
    } // run tests
    stage('Code Coverage') {
        when {
            expression { return params.WITH_CODE_COVERAGE }
        }
        steps {
            echo "Running code covarage..."
            sh """#!/usr/bin/env bash
            source /nuenv.sh
            set -eux
            pushd nucore/
            make -j8 code-coverage-build
            popd
            """
            sh """#!/usr/bin/env bash
            source /nuenv.sh
            set -eux
            pushd nucore/
            make code-coverage
            popd
            """
        }
    }
    stage('Generate Docs') {
        steps {
            timeout(time: 10, unit: 'MINUTES') {
                sh """#!/usr/bin/env bash
                    source /nuenv.sh
                    doxygen Doxyfile
                """
            }
        }
    }
    stage('Run Simulations') {
        steps {
            echo "Running Simulations..."
            sh """#!/usr/bin/env bash
                source /nuenv.sh
                set -eux
                # simulation test
                free -m
                export LOG_LOCATION=\$(date +%F-%H-%M-%S)
                cd \${NUCORE}/nucore/tests/scenario-driver && python test_point_on_baseline.py
                ./autotest.py --runprefix \${LOG_LOCATION} scenarios/scn_cleantech_car003_gotogoal.json && ./review.py
                ../regression/metrics.py logs/\${LOG_LOCATION}/scn_cleantech_car003_gotogoal
                ./autotest.py scenarios/scn_onenorth_car004_gotogoal.json
                ./autotest.py scenarios/scn_onenorth_car005_gotogoal.json
                ./autotest.py scenarios/scn_boston_car006_gotogoal.json
                ./autotest.py scenarios/scn_boston_car008_gotogoal.json
            """
        }
    }

} // stages

post {
    always {
        sh """git clean -fdx"""
    }
    aborted {
        slackSend color: "#edb612", message: """Aborted ${env.JOB_NAME} #${env.BUILD_NUMBER} [${env.CHANGE_AUTHOR_DISPLAY_NAME}] (<${env.BUILD_URL}|Open>)
${env.CHANGE_BRANCH}: ${env.CHANGE_TITLE}"""
    }
    failure {
        slackSend color: "#c61515", message: """Failed ${env.JOB_NAME} #${env.BUILD_NUMBER} [${env.CHANGE_AUTHOR_DISPLAY_NAME}] (<${env.BUILD_URL}|Open>)
${env.CHANGE_BRANCH}: ${env.CHANGE_TITLE}"""
    }
    //changed {
        // only run if the current Pipeline run has a different status from previously completed Pipeline
    //}
} // post

} // pipeline
