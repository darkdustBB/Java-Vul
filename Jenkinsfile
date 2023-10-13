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
                   semgrep scan --config auto | tee /root/.jenkins/workspace/Devsecops3/semgrep_output_1
                '''
            }
        }
  stage('Dynamic application security testing'){
            steps{
                sh '''
                     skipfish -o /opt/skipfishoutput_new_output_out_out_p http://172.16.1.23:1337
                     zip -r skipfishoutput6.zip /opt/skipfishoutput_new_output_out_out_p
                     pwd
                '''
            }
        }


        stage('Container image testing with trivy'){
            steps{
                sh '''
                    trivy image dvwa-dvwa | tee trivy_output
                '''     
        }
        }

        stage('Container image testing with Anchore-engine'){
            steps{
                sh '''
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar image add docker.io/library/debian:latest
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar image wait docker.io/library/debian:latest
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar image list
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar image get docker.io/library/debian:latest
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar system feeds list
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar system wait
                    anchore-cli --url http://localhost:8228/v1 --u admin --p foobar image vuln docker.io/library/debian:latest os | tee Anchore_output
                    
                '''     
        }
        }

          stage('Container image testing with grype'){
            steps{
                sh '''
                    grype dvwa-dvwa | tee grype_output
                '''     
        }
        }


         stage('Container image testing with clair'){
            steps{
                sh '''
                     clair-scanner --clair=http://172.16.1.204:6060 --ip=localhost vulnerable-java-application_vulnado| tee clair_output
                '''     
        }
        }
       
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

     stage('Debug') {
    steps {
        sh 'ls -R'
        sh 'pwd'
    }
}

        stage('Archive Zip Folder') {
    steps {
        archiveArtifacts artifacts: 'skipfishoutput6.zip', allowEmptyArchive: true
    }
}


    }

       post {
        always {
            // Archive the results as a build artifact
            archiveArtifacts 'detect-secrets_output'
            archiveArtifacts 'semgrep_output_1'
            archiveArtifacts 'trivy_output' 
            archiveArtifacts 'Anchore_output' 
            archiveArtifacts 'grype_output'
            archiveArtifacts 'clair_output'

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
