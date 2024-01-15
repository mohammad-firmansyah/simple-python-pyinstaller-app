node {
    stage('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        
        junit 'test-reports/results.xml'
    }

     stage('Manual Approval') {
         input message: 'Sudah selesai menggunakan Python App? (Klik "Proceed" untuk mengakhiri)', ok: 'Proceed'
    }
    
    stage('Deliver') {
        def VOLUME = "${pwd()}/sources:/src"
        def IMAGE = 'cdrx/pyinstaller-linux:python2'
        unstash(name: 'compiled-results')
        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        archiveArtifacts "sources/dist/add2vals"
        sleep time: 60 , unit: 'SECONDS'

        } 
    
  
}
