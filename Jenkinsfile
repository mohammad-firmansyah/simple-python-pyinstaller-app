node {
    // Skip stages after an unstable result
    currentBuild.result = 'SUCCESS'

    // Build stage
    stage('Build') {
        // Use Python 3.12.1-alpine3.19 Docker image
        docker.image('python:3.12.1-alpine3.19').inside {
            // Compile Python files and stash the results
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    // Test stage
    stage('Test') {
        // Use qnib/pytest Docker image
        docker.image('qnib/pytest').inside {
            // Run pytest and generate JUnit XML report
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        // Always publish JUnit test results
        junit 'test-reports/results.xml'
    }

    // Deliver stage
    stage('Deliver') {
        // Run on any available agent
        agent any
        // Define environment variables
        environment {
            VOLUME = '$(pwd)/sources:/src'
            IMAGE = 'cdrx/pyinstaller-linux:python2'
        }
        // Execute steps inside a directory
        dir(path: env.BUILD_ID) {
            // Unstash the compiled results and build the deliverable
            unstash(name: 'compiled-results')
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }
    }

    // Post steps for success
    stage('Post-Deliver') {
        // Archive the deliverable and clean up
        archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
    }
}
