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

    environment {
      PATH="/var/lib/jenkins/miniconda3/bin:$PATH"
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
                sh ''' echo ${SHELL}
                       [-d venv ] && rm -rf venv
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

        stage ("Extract test results") {
              cobertura coberturaReportFile: 'reports/coverage.xml'
          }



        stage('Unit tests') {
            steps {
                sh  ''' . venv/bin/activate
                        python -m pytest --verbose --junit-xml reports/unit_tests.xml
                    '''
            }


        }

        stage ("Extract Unit test results") {
              cobertura coberturaReportFile: 'reports/unit_tests.xml'
          }



        // stage("Deploy to PyPI") {
        //     steps {
        //         sh """twine upload dist/*
        //         """
        //     }
        // }
    }


}
