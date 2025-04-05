pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                bat 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        stage('Test') {
            steps {
                // Create test-reports directory if it doesn't exist
                bat 'if not exist test-reports mkdir test-reports'

                // Show the files in sources to confirm test file is there
                bat 'echo Checking sources directory:'
                bat 'dir sources'

                // Install pytest
                bat 'pip install pytest'

                // Run pytest and generate JUnit-style test report
                bat 'python -m pytest --junit-xml=test-reports/results.xml sources/test_calc.py'

                // Show the contents of the test-reports directory for debugging
                bat 'echo Contents of test-reports directory:'
                bat 'dir test-reports'
            }
            post {
                always {
                    script {
                        if (fileExists('test-reports/results.xml')) {
                            echo '✅ Test results found, publishing JUnit report...'
                            junit 'test-reports/results.xml'
                        } else {
                            echo '⚠️ Test report not found. Skipping junit results step.'
                        }
                    }
                }
            }
        }

        stage('Deliver') {
            steps {
                // Create a standalone executable with PyInstaller
                bat 'pyinstaller --onefile sources/add2vals.py'
            }
            post {
                success {
                    // Archive the generated binary file
                    archiveArtifacts 'dist/add2vals.exe'
                }
            }
        }
    }
}
