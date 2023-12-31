node {
    stage('Build') {
        try {
            // Your existing build steps
            echo 'Building...'
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        } catch (Exception e) {
            currentBuild.result = 'UNSTABLE'
            error('Build failed!')
        }
    }

    stage('Test') {
        try {
            // Your existing test steps
            echo 'Testing...'
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        } catch (Exception e) {
            currentBuild.result = 'UNSTABLE'
            error('Test failed!')
        } finally {
            junit 'test-reports/results.xml'
        }
    }

    stage('Deliver') {
        try {
            // Your existing deliver steps
            echo 'Delivering...'
            dir(path: env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
            }
        } catch (Exception e) {
            currentBuild.result = 'UNSTABLE'
            error('Deliver failed!')
        } finally {
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }
    }
}
