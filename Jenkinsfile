pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/re-r00/abcd-student', branch: 'main'
                }
            }
        }    
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('SCA') {
            steps {
                sh '''
                    docker rm -f juice-shop || true
                
                    docker run --name juice-shop -d \
                    -p 3000:3000 \
                    bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    osv-scanner scan --lockfile package-lock.json > results/sca_report.txt || true
                '''
            }
            post {
                always {
                    echo 'Archiving results...'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
    }
    post {
    always {
        echo 'Archiving results...'
        archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        echo 'Sending SCA reports to DefectDojo...'
        defectDojoPublisher(
            artifact: 'results/sca_report.txt',  
            productName: 'Juice Shop',            
            scanType: 'SCA - Dependency Scan',          
            engagementName: 'jakub.kolataj@xtb.com')
        }
    }
}
