// // Membuat Jenkins Pipeline dengan Scripted Pipeline
// // Mulai dengan mendefinisikan pipeline
// node {
//     // Untuk mendefinisikan sebuah stage (tahapan) 'Build' untuk melakukan proses kompilasi kode Python
//     stage('Build') {
//         // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan Menggunakan Docker image python:2-alpine sebagai lingkungan untuk menjalankan stage 'Build'
//         docker.image('python:2-alpine').inside {
//             // Menjalankan perintah shell untuk mengkompilasi file Python add2vals.py dan calc.py
//             sh 'python -m py_compile sources/add2vals.py sources/calc.py'
//         }
//     }
//     // Untuk mendefinisikan sebuah stage (tahapan) 'Test' untuk menjalankan tes menggunakan pytest
//     stage('Test') {
//         // // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan Menggunakan Docker image qnib/pytest sebagai lingkungan untuk menjalankan stage 'Test'
//         docker.image('qnib/pytest').inside {
//             // Menjalankan perintah shell untuk menjalankan tes dan menghasilkan laporan JUnit XML
//             sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
//         }
//     }
// }

pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
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
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
       stage('Deliver') {
            steps {
                script {
                    // Gunakan docker.image().inside() pattern
                    docker.image('cdrx/pyinstaller-linux:python2').inside('--privileged') {
                        sh '''
                            echo "Current directory:"
                            pwd
                            echo "Directory contents:"
                            ls -la
                            echo "Python version:"
                            python --version
                            
                            pyinstaller --onefile sources/add2vals.py
                        '''
                    }
                }
                
                // Archive artifacts setelah keluar dari container
                archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
            }
        }
    }
}
