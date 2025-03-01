// Membuat Jenkins Pipeline dengan Scripted Pipeline
// Mulai dengan mendefinisikan pipeline
node { 
    // Untuk mendefinisikan sebuah stage (tahapan) 'Build' untuk melakukan proses kompilasi kode Python
    stage('Build') {
        // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan Menggunakan Docker image python:2-alpine sebagai lingkungan untuk menjalankan stage 'Build'
        docker.image('python:2-alpine').inside {
             // Menjalankan perintah shell untuk mengkompilasi file Python add2vals.py dan calc.py
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            // Menyimpan hasil kompilasi untuk digunakan di stage selanjutnya
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    // Untuk mendefinisikan sebuah stage (tahapan) 'Test' untuk menjalankan tes menggunakan pytest
    stage('Test') {
        // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan Menggunakan Docker image qnib/pytest sebagai lingkungan untuk menjalankan stage 'Test't
        docker.image('qnib/pytest').inside {
            // Menjalankan perintah shell untuk menjalankan tes dan menghasilkan laporan JUnit XML
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
    }

    // Untuk mendefinisikan sebuah stage (tahapan) 'Manual Approval' untuk pengguna bisa memilih apakah klik Proceed (melanjutkan eksekusi pipeline ke tahap Deploy) atau Abort (menghentikan eksekusi pipeline)
    stage('Manual Approval') {
        // Menlankan perintah input message untuk lanjut ke tahap deploy atau menghentikan eksekusi
        input message: 'Lanjutkan ke tahap Deliver?'
    }

    // Untuk mendefinisikan sebuah stage (tahapan) 'Deploy' untuk membuat executeable file dan mengirimnya ke ec2
    stage('Deploy') {
        withCredentials([string(credentialsId: 'ec2-password', variable: 'EC2_PASSWORD')]) {
        // Mendefinisikan environment variables
        def VOLUME = '$(pwd)/sources:/src'
        def IMAGE = 'cdrx/pyinstaller-linux:python2'
        def EC2_HOST = '54.151.249.134'
        def EC2_USER = 'ec2-user' 

            // Mengambil kembali hasil kompilasi yang telah di-stash
            unstash(name: 'compiled-results')

            // Membuat executable menggunakan PyInstaller
            sh "docker run --rm --user root -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
            
            // Menambahkan pengecekan keberadaan file dist
            sh "ls -la sources/dist"

            // Membuat file arsip terkompresi dari sources/dist/ add2vals
            sh "tar -czf add2vals.tar.gz -C sources/dist/ add2vals"
            
            // Membuat artifact
            archiveArtifacts "add2vals.tar.gz"

            // Mengirim file ke EC2
            sh """
                docker run --rm -v \$(pwd):/src --user root ubuntu bash -c '
                    apt-get update && 
                    apt-get install -y sshpass openssh-client && 
                    sshpass -p "${EC2_PASSWORD}" scp -o StrictHostKeyChecking=no /src/add2vals.tar.gz ${EC2_USER}@${EC2_HOST}:~/'
                    tar -xzf add2vals.tar.gz
                    ./add2vals 5 3
            """

            // mengekstrak (unpack) isi dari file terkompresi bernama add2vals.tar.gz
            sh "tar -xzf add2vals.tar.gz"

            // menjalankan program bernama add2vals dengan dua argumen: 5 dan 3
            sh "./add2vals 5 3"

            // Menjalankan perintah sleep untuk menjeda eskekusi pipeline agar aplikasi bisa tetap berjalan selama 1 menit.
            sleep time: 1, unit: 'MINUTES'

            // untuk menampilkan pesan status deployment.
            echo "Deployment selesai, Berhasil mengirim ke http://${EC2_HOST}"
        }
    }
}