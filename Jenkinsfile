pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * 1-5')
    }

    options {
        skipDefaultCheckout(true)
        // Keep the 10 most recent builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {

        stage ("Code pull"){
            steps{
                checkout scm
            }
        }

        stage('Build environment') {
            steps {
              echo "Building virtualenv"
              sh ''' echo ${SHELL} [-d venv ] && rm -rf venv
                     virtualenv venv
                     . venv/bin/activate
                     pip install -r requirements/dev.txt
                           '''
                    }
        }

        stage('Static code metrics') {
            steps {
                echo "Test coverage"
                sh  ''' . venv/bin/activate
                        coverage run irisvmpy/iris.py 1 1 2 3
                        python -m coverage xml -o reports/coverage.xml
                    '''
                echo "Style check"
                sh  ''' . venv/bin/activate
                        pylint irisvmpy || true
                    '''
            }

        }

        stage('Unit tests') {
            steps {
                sh  ''' . venv/bin/activate
                        python -m pytest --verbose --junit-xml reports/unit_tests.xml
                    '''
            }

  
        }
    }


}
