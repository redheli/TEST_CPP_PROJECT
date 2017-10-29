pipeline {

agent {
    docker {
         image "ubuntu:16.04"
         args "-u root"
    }
}
stages {
    stage('Build All') {
        steps {
            echo "Building ${env.BUILD_DIR}"
            sh """#!/usr/bin/env bash
                set -eux
		echo "running whoami" & whoami
		echo "running groups" & groups
		echo $PWD
		echo "ls" && ls
		echo "ls build" && ls -la build
                mkdir -p build
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
