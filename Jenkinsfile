// Membuat Jenkins Pipeline dengan Scripted Pipeline
// Mulai dengan mendefinisikan pipeline
pipeline {
    agent none
    
    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                    args '-u root'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                    args '-u root'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }
        
        stage('Deploy') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    echo "Current directory: $PWD"
                    echo "Listing directory contents:"
                    ls -la
                    
                    echo "Installing PyInstaller..."
                    pip install --upgrade pip
                    pip install pyinstaller
                    
                    echo "Building executable..."
                    pyinstaller --onefile sources/add2vals.py
                    
                    echo "Listing dist directory:"
                    ls -la dist/
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
                }
            }
        }
    }
}

// pipeline {
//     agent none
//     stages {
//         stage('Build') {
//             agent {
//                 docker {
//                     image 'python:2-alpine'
//                 }
//             }
//             steps {
//                 sh 'python -m py_compile sources/add2vals.py sources/calc.py'
//             }
//         }
//         stage('Test') {
//             agent {
//                 docker {
//                     image 'qnib/pytest'
//                 }
//             }
//             steps {
//                 sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
//             }
//             post {
//                 always {
//                     junit 'test-reports/results.xml'
//                 }
//             }
//         }
//         stage('Deliver') {
//             agent {
//                 docker {
//                     image 'cdrx/pyinstaller-linux:python2'
//                 }
//             }
//             steps {
//                 sh '''
//                     cd sources
//                     python add2vals.py
//                  '''
//             }
//             post {
//                 success {
//                     archiveArtifacts 'dist/add2vals'
//                 }
//             }
//         }
//     }
// }
