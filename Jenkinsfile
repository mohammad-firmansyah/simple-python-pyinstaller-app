node {
    stage('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            // Run pytest and generate JUnit XML report
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        // Always publish JUnit test results
        junit 'test-reports/results.xml'
    }

    stage('Deliver') {
        def VOLUME = "${pwd()}/sources:/src"
        def IMAGE = 'cdrx/pyinstaller-linux:python2'
        
        input message: 'Sudah selesai menggunakan Python App? (Klik "Proceed" untuk mengakhiri)', submitter: 'user'
        
        timeout(time: 5, unit: 'SECONDS') {
            catchError {
                dir("${BUILD_ID}") {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
        }

         // A simple step using 'echo'
        echo 'Hello, World!'

        // Another step using 'sh' (shell)
        sh 'ls -la'

        // A step using a custom script
        script {
            def result = myCustomFunction()
            echo "Custom function result: ${result}"
        }


        post {
            success {
                archiveArtifacts "${BUILD_ID}/sources/dist/add2vals"
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            }
        }
    }
}

def myCustomFunction() {
    // Custom logic goes here
    return "Custom result"
}
