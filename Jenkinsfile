// Scripted Pipeline for simple-python-pyinstaller-app
// Build -> Test -> Manual Approval -> Deploy
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

    // Continuous delivery: a person decides whether this build goes to Deploy.
    // "Proceed" continues to the Deploy stage, "Abort" stops the pipeline.
    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
    }

    stage('Deploy') {
        // Full python image: PyInstaller needs binutils (objdump), which the alpine image lacks.
        docker.image('python:3.12').inside {
            withEnv(["HOME=${env.WORKSPACE}"]) {
                // Bundle add2vals into a standalone Linux executable
                sh 'python -m venv .venv-deploy'
                sh '.venv-deploy/bin/pip install --no-cache-dir --quiet pyinstaller'
                sh '.venv-deploy/bin/pyinstaller --onefile --clean --distpath dist --workpath build sources/add2vals.py'

                // Smoke-test the deployed executable
                sh './dist/add2vals 10 20'
                sh './dist/add2vals Dicoding Indonesia'

                archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true

                echo 'Aplikasi berhasil di-deploy. Aplikasi akan berjalan selama 1 menit...'
                sleep time: 1, unit: 'MINUTES'
                echo 'Waktu 1 menit telah berakhir. Aplikasi dihentikan, pipeline selesai.'
            }
        }
    }
}
