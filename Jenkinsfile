pipeline {
    agent {
        dockerfile {
            label 'slave'
        }
    }

    stages {
        stage('Build') {
            steps {
                sh 'eval `opam config env` && autoreconf && mkdir build && cd build && ../configure && make'
            }
        }
        stage('Test') {
            steps {
                script {
                    def switches = sh (
                        script: 'opam switch -i -s',
                        returnStdout: true
                    ).split('\n')
                    def branches = [:]
                    for (i in switches) {
                        def switch_name = i
                        branches[switch_name] = {
                            sh "opam switch $switch_name && eval `opam config env` && mkdir build/$switch_name && cd build/$switch_name && ../../configure && make && make tests"
                        }
                    }
                    parallel branches
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'cd build && make dist'
                archiveArtifacts artifacts: 'build/*.tar.gz', fingerprint: true
            }
        }
    }
}
