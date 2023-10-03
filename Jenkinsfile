pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    sh """
                        echo "PATH = ${PATH}"
                        echo "M2_HOME = ${M2_HOME}"
                    """
                }
            }
        }

        stage('Scan Code with Detect-Secrets') {
            steps {
                sh '''
                detect-secrets -C /opt/Java-Vul scan | tee detect-secrets_output
                '''
            }
        }


        stage('Static application security testing'){
            steps{
                sh '''
                   cd /opt/Java-Vul
                   semgrep scan --config auto | tee Semgrep_Output
                '''
            }
        }
  stage('Dynamic application security testing'){
            steps{
                sh '''
                     skipfish -o /opt/result_skipfish_fresh  http://172.16.1.23:1337
                '''
            }
        }
       
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }

       post {
        always {
            // Archive the Trufflehog results as a build artifact
            archiveArtifacts 'detect-secrets_output'
            archiveArtifacts '/opt/Java-Vul/Semgrep_Output'
            
            // Publish HTML report
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '/opt/result_skipfish_fresh', // Change this to the correct directory
                reportFiles: 'index.html',    // Change this to the correct report file
                reportName: 'Skipfish Report',
                reportTitles: 'Skipfish Report'
            ])
            
            

             // Send email notifications
        emailext(
            subject: "Build ${currentBuild.result}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: """
            <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            <p>Git Secrets Report: <a href="${env.BUILD_URL}/artifact/secrets.txt">secrets.txt</a></p>
            <p>SAST Output: <a href="${env.BUILD_URL}/artifact/SAST_output.txt">SAST_output.txt</a></p>
            <p>ZAP Report: <a href="${env.BUILD_URL}/artifact/output_ZAP.html">output_ZAP.html</a></p>
            """,
            to: "bluedustbb@gmail.com", // Add the recipient email address here
            attachLog: true, // Attach build log
            compressLog: true // Compress build log
        )
           
        }
    }
}
