pipeline {
    agent any
    
    tools{
        jdk   'jdk'
        maven   'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git check-out') {
            steps {
               git branch: 'main', url: 'https://github.com/kasireddysairam/Boardgame.git'
            }
        }
   
         stage('compile') {
            steps {
                sh " mvn compile"
               
            }
        }
        
             stage('test') {
               steps {
                sh  "mvn test"
               }
               }
            stage('file system scan') {
            steps {
               sh "trivy fs --format table -o trivy-fs-report.html ."
               
            }
        }
           stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
               sh '''  $SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectKey=BoardGame  -Dsonar.projectName=BoardGame \
                -Dsonar.java.binaries=. '''

}
            }
        }
 
        
        
         stage('build') {
            steps {
                sh "mvn package"
               
            }
        }
        
        
         stage('Publish to Nexus') {
            steps {
          withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
              sh  "mvn deploy"
           }
            }
             }
             
             stage('Build and Tag Docker Image') {
              steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { 
  
                   sh "docker build -t sairamk1998/boardgame:latest ."
               }
                }
                
                }
                
             }
             
              
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html sairamk1998/boardshack:latest "
            }
        }
          
          
            
             stage('push docker image to docker hub') {
              steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
  
                   sh "docker push sairamk1998/boardgame:latest"
               }
                }
            }
        }
       stage('Deploy To Kubernetes') {
            steps {
                 withKubeConfig(caCertificate: '', clusterName: 'bmsprojectcluster.ap-south-1.eksctl.io', contextName: '', credentialsId: 'k8-creds', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://DDC31A136275C4C39B522193561BB17E.gr7.ap-south-1.eks.amazonaws.com') {
 
                  sh "kubectl apply -f deployment-service.yaml"
           }
            }
        }

           stage('Verify the Deployment') {
            steps {

                withKubeConfig(caCertificate: '', clusterName: 'bmsprojectcluster.ap-south-1.eksctl.io', contextName: '', credentialsId: 'k8-creds', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://DDC31A136275C4C39B522193561BB17E.gr7.ap-south-1.eks.amazonaws.com') {

                sh "kubectl get pods -n webapps"
                sh "kubectl get svc -n webapps"
           }
            }
            }
         }








         post {
        always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'kasireddysairam03@gmail.com',
                from: 'kasireddysairam03@example.com',
                replyTo: 'kasireddysairam03@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

}
