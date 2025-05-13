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
                    git credentialsId: 'github-token', url: 'https://github.com/urbanski717/abcd-student', branch: 'main'
                }
            }
        }
        stage("[omv] SKAN"){
            steps{
                sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json'
            }
            post{
                always{
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
        
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /mnt/c/Users/Kamil/Desktop/Projekty/abcmount:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c "\
                        zap.sh -cmd -addonupdate;\
                        zap.sh -cmd -addoninstall communityScripts\
                        -addoninstall pscanrulesAlpha\
                        -addoninstall pscanrulesBeta\
                        -autorun /zap/wrk/passive_scan.yaml" \
                        || true
                '''   
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                        docker stop zap juice-shop || true
                        docker rm zap || true
                    '''
                }
            }
        }
    }
}

