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
                     skipfish -o /opt/skipfish  http://172.16.1.23:1337
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

        stage('Publish HTML Folder') {
    steps {
        archiveArtifacts allowEmptyArchive: true, artifacts: 'opt/skipfish', onlyIfSuccessful: true
    }
}


    }


       post {
        always {
            // Archive the Trufflehog results as a build artifact
            archiveArtifacts 'detect-secrets_output'
            archiveArtifacts 'semgrep_output_1'
            
          
            
            

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
