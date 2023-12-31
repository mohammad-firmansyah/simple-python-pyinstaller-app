node {
    options {
        skipStagesAfterUnstable()
    }

    stage('Build') {
        agent {
            docker {
                image 'python:3.12.1-alpine3.19'
            }
        }
        steps {
            script {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash name: 'compiled-results', includes: 'sources/*.py*'
            }
        }
    }

    stage('Test') {
        agent {
            docker {
                image 'qnib/pytest'
            }
        }
        steps {
            script {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }
        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }

    stage('Deliver') {
        agent any
        environment {
            VOLUME = '$(pwd)/sources:/src'
            IMAGE = 'cdrx/pyinstaller-linux:python2'
        }
        steps {
            script {
                dir(path: env.BUILD_ID) {
                    unstash name: 'compiled-results'
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
        }
        post {
            success {
                script {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}
