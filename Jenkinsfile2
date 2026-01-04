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
                sh '''
                    sudo service apache2 start
                '''
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
                    mkdir -p report

                    echo "<html><body><h1>Apache Log Report</h1>" > report/report.html

                    echo "<h2>4xx errors</h2><pre>" >> report/report.html
                    sudo grep ' 4[0-9][0-9] ' /var/log/apache2/access.log \
              | tee report/4xx.log \
              >> report/report.html || echo "No 4xx errors" >> report/report.html
                    echo "</pre>" >> report/report.html

                    echo "<h2>5xx errors</h2><pre>" >> report/report.html
                    COUNT_5XX=$(sudo grep ' 5[0-9][0-9] ' /var/log/apache2/access.log | tee report/5xx.log | wc -l)

                    if [ "$COUNT_5XX" -gt 0 ]; then
                        echo "5xx errors found: $COUNT_5XX"
                        echo "5xx errors found: $COUNT_5XX" >> report/report.html
                        echo "</pre></body></html>" >> report/report.html
                        exit 1
                    else
                        echo "No 5xx errors found"
                        echo "No 5xx errors found" >> report/report.html
                        echo "</pre></body></html>" >> report/report.html
                    fi
                 '''
            }
        }
    }

    post {
        always {
            echo "Archiving logs and report..."
            archiveArtifacts artifacts: 'report/**/*', fingerprint: true
        }

        failure {
            echo "Pipeline FAILED due to 5xx errors"
        }
    }
}
