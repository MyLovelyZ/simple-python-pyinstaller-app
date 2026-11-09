// Scripted Pipeline for simple-python-pyinstaller-app
node {
    // Poll SCM: check the Git repository every 2 minutes
    properties([
        pipelineTriggers([
            pollSCM('H/2 * * * *')
        ])
    ])

    stage('Checkout') {
        checkout scm
    }

    docker.image('python:3.12-alpine').inside {
        stage('Build') {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }

        stage('Test') {
            withEnv(["HOME=${env.WORKSPACE}"]) {
                try {
                    sh 'pip install --user --no-cache-dir pytest'
                    sh 'python -m pytest --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
                } finally {
                    junit 'test-reports/results.xml'
                }
            }
        }
    }
}
