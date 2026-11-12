pipeline {
    agent {
        label 'Node-1'
    }

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout Source') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    set -eu
                    echo "Running on: $NODE_NAME"
                    whoami
                    hostname
                    ansible --version
                    sudo -n true
                '''
            }
        }

        stage('Check Playbook Syntax') {
            steps {
                sh '''
                    ansible-playbook \
                      -i localhost, \
                      -e ansible_python_interpreter=/usr/bin/python3 \
                      deploy.yml --syntax-check
                '''
            }
        }

        stage('Deploy Webpage') {
            steps {
                sh '''
                    ansible-playbook \
                      -i localhost, \
                      -e ansible_python_interpreter=/usr/bin/python3 \
                      deploy.yml
                '''
            }
        }

        stage('Verify Exact Deployed File') {
            steps {
                sh '''
                    set -eu
                    curl --fail --silent --show-error \
                      http://127.0.0.1/ -o deployed-page.html

                    cmp index.html deployed-page.html
                    echo "The served webpage matches this build's source."
                '''

                archiveArtifacts artifacts: 'deployed-page.html',
                    fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Ansible deployed and verified the webpage.'
        }

        failure {
            echo 'FAILED: Check the first error in Console Output.'
        }
    }
}
