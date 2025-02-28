// Membuat Jenkins Pipeline dengan Scripted Pipeline
node {
    // Mendefinisikan pipeline Jenkins
        def EC2_HOST = '54.151.249.134'
        def EC2_USER = 'ec2-user'  
        // Stage 'Build': Tahap kompilasi kode Python
        stage('Build') {
            // Menggunakan Docker untuk menjalankan build
            docker.image('python:2-alpine').inside {
                // Mengkompilasi file Python menjadi bytecode (.pyc)
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                // Menyimpan hasil kompilasi untuk digunakan di stage selanjutnya
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        // Stage 'Test': Tahap pengujian kode
        stage('Test') {
            // Menggunakan Docker untuk menjalankan test
            docker.image('qnib/pytest').inside {
                // Menjalankan pytest dan menghasilkan laporan dalam format XML
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
                // Publikasikan hasil test menggunakan plugin JUnit
                junit 'test-reports/results.xml'
            }
        }

        // Stage 'Manual Approval': Tahap persetujuan manual
        stage('Manual Approval') {
            // Menambahkan tahap persetujuan manual seperti di Jenkinsfilereact
            input message: 'Lanjutkan ke tahap Deliver?'
        }

        // Stage 'Deliver': Tahap pembuatan executable
        stage('Deliver') {
            withCredentials([string(credentialsId: 'ec2-password', variable: 'EC2_PASSWORD')]) {
            // Mendefinisikan environment variables
            def VOLUME = '$(pwd)/sources:/src'
            def IMAGE = 'cdrx/pyinstaller-linux:python2'
            
            // Membuat direktori dengan nama BUILD_ID
            // dir(path: env.BUILD_ID) {
                // Mengambil kembali hasil kompilasi yang telah di-stash
                    unstash(name: 'compiled-results')

                    sh """
                        if [ -d "sources/dist" ]; then
                            echo "File dist berhasil dibuat"
                     else
                            echo "File dist tidak ditemukan"
                        fi
                    """
                // Membuat executable menggunakan PyInstaller
                    sh "docker run --rm --user root -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    // sh 'pyinstaller --onefile sources/add2vals.py'
                    
                
                // Menambahkan pengecekan keberadaan file dist
                    sh "ls -la sources/dist"

                    sh "tar -czf add2vals.tar.gz -C sources/dist/ add2vals"
                    
                    archiveArtifacts "add2vals.tar.gz"

                    sh """
                        docker run --rm -v \$(pwd):/src --user root ubuntu bash -c '
                            apt-get update && 
                            apt-get install -y sshpass openssh-client && 
                            sshpass -p "${EC2_PASSWORD}" scp -o StrictHostKeyChecking=no /src/add2vals.tar.gz ${EC2_USER}@${EC2_HOST}:~/'
                    """
                    sh "tar -xzf ~/add2vals.tar.gz -C"

                    sh "./add2vals 5 3"

                    echo "Deployment selesai, aplikasi tersedia di http://${EC2_HOST}"
            // }
            }
            
            // Jika berhasil, arsipkan hasil build dan bersihkan direktori
            // if (currentBuild.result == 'SUCCESS') {
            //     archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            //     sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            // }
        }
}