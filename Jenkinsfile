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
        stage('DAST') {
            steps {
                sh '''
                    docker rm -f juice-shop || true
                    docker run --name juice-shop -d \
                    -p 3000:3000 \
                    bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker rm -f zap || true
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v "C:/Users/kkuuu/OneDrive/Pulpit/ABCdevsecops/abcd-student/.zap:/zap/wrk/:rw" \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                    || true
                '''
            }
            post {
                always {
                    // ${{WORKSPACE}} resolves to /var/jenkins_home/workspace/ABCD
                    sh '''
                        sleep 10
                        docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap
                        '''
                }
            }
        }
    }
} 
