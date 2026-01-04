pipeline {
    agent any

    stages {

        stage('Check Operating System') {
            steps {
                echo "Checking operating system..."
                sh 'cat /etc/os-release'
            }
        }

        stage('Install Apache2') {
            steps {
                echo "Installing Apache2..."
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y apache2 curl
                '''
            }
        }

        stage('Start Apache2') {
            steps {
                echo "Starting Apache2..."
                sh 'sudo service apache2 start'
            }
        }

        stage('Generate Test Requests') {
            steps {
                echo "Generating HTTP requests (200 and 404)..."
                sh '''
                    curl -s http://localhost > /dev/null
                    curl -s http://localhost/notfound > /dev/null || true
                '''
            }
        }

        stage('Analyze Apache Logs') {
            steps {
                echo "Analyzing Apache logs..."
                sh '''
                    # Створюємо report без sudo, щоб Jenkins міг архівувати
                    mkdir -p report
                    ACCESS_LOG=/var/log/apache2/access.log

                    echo "<html><body><h1>Apache Log Report</h1>" > report/report.html

                    # 4xx
                    echo "<h2>4xx errors</h2><pre>" >> report/report.html
                    sudo grep ' 4[0-9][0-9] ' $ACCESS_LOG > report/4xx.log 2>/dev/null \
                      || echo "No 4xx errors" > report/4xx.log
                    cat report/4xx.log >> report/report.html
                    echo "</pre>" >> report/report.html

                    # 5xx
                    echo "<h2>5xx errors</h2><pre>" >> report/report.html
                    sudo grep ' 5[0-9][0-9] ' $ACCESS_LOG > report/5xx.log 2>/dev/null \
                      || echo "No 5xx errors" > report/5xx.log
                    cat report/5xx.log >> report/report.html

                    COUNT_5XX=$(grep -c ' 5[0-9][0-9] ' report/5xx.log || echo 0)

                    echo "</pre></body></html>" >> report/report.html

                    if [ "$COUNT_5XX" -gt 0 ]; then
                        echo "5xx errors found: $COUNT_5XX"
                        exit 1
                    else
                        echo "No 5xx errors found"
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "Archiving logs and report..."
            archiveArtifacts artifacts: 'report/*', allowEmptyArchive: true, fingerprint: true
        }

        failure {
            echo "Pipeline FAILED due to 5xx errors"
        }
    }
}
