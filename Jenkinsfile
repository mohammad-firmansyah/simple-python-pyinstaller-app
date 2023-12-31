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

}
