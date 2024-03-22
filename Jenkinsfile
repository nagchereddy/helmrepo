//def ARTIFACT_CREDS 
pipeline{
    agent any
    environment{
        REPO_CREDS = 'github'
        ARTIFACT_CREDS = credentials('jenkinsogcp')
        GITHUB_NAME = 'gcprepo'
        GCR_URL = 'us-east1-docker.pkg.dev/solid-antler-409714/gcprepo'
        APP_NAME = 'httpd'
    }
    stages{
        stage("Checkout code"){
            steps{
                checkout scm
            }
            
        }
        stage("authenticating with GCP"){
            steps{
                script{
                withCredentials([file(credentialsId: 'jenkinsogcp', variable: 'ARTIFACT_CREDS')]) {
                        sh("gcloud  auth activate-service-account --key-file=${ARTIFACT_CREDS}")
                        def customImage = docker.build("${GCR_URL}/${APP_NAME}:${env.BUILD_ID}")

                        /* Push the container to the custom Registry */
                        customImage.push()
                        
            }
            }
            }
        }
        // stage("Building Docker image"){
        //     steps{
        //         script{
        //             // docker.withRegistry("${GCR_URL}", "${ARTIFACT_CREDS}") {

        //                 def customImage = docker.build("${GCR_URL}/${APP_NAME}:${env.BUILD_ID}")

        //                 /* Push the container to the custom Registry */
        //                 customImage.push()
        //             // }
        //         }
        //     }
        // }
}
}