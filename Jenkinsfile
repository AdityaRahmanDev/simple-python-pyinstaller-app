// Membuat Jenkins Pipeline dengan Scripted Pipeline
// Mulai dengan mendefinisikan pipeline
node {
    // Untuk mendefinisikan sebuah stage (tahapan) 'Build' untuk melakukan proses kompilasi kode Python
    stage('Build') {
        // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan Menggunakan Docker image python:2-alpine sebagai lingkungan untuk menjalankan stage 'Build'
        docker.image('python:2-alpine').inside {
            // Menjalankan perintah shell untuk mengkompilasi file Python add2vals.py dan calc.py
            sh 'python -m py_compile /home/OneDrive/Documents/LabDicoding/simple-python-pyinstaller-app/sources/add2vals.py /home/OneDrive/Documents/LabDicoding/simple-python-pyinstaller-app/sources/calc.py'
        }
    }

    // Untuk mendefinisikan sebuah stage (tahapan) 'Test' untuk menjalankan tes menggunakan pytest
    stage('Test') {
        // // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan Menggunakan Docker image qnib/pytest sebagai lingkungan untuk menjalankan stage 'Test'
        docker.image('qnib/pytest').inside {
            // Menjalankan perintah shell untuk menjalankan tes dan menghasilkan laporan JUnit XML
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        // Setelah stage 'Test' selesai akan menjalankan Post-action terlepas apakah stage 'Test' berhasil (success) atau gagal (failure)
        post {
            always {
                // Post-action yang selalu dijalankan setelah stage 'Test' selesai akan mempublikasikan laporan JUnit XML.
                junit 'test-reports/results.xml'
            }
        }
    }
}