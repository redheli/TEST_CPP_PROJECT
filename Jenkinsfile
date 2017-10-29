pipeline {

agent {
    docker "ubuntu:16.04"
}
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

} // stages

post {
    	always 
	{
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
} // post

} // pipeline
