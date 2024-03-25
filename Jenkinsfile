//def ARTIFACT_CREDS 
pipeline{
    agent any
    environment{
        REPO_CREDS = 'github'
        ARTIFACT_CREDS = credentials('jenkinsogcp')
        GITHUB_NAME = 'gcprepo'
        GCR_URL = 'us-east1-docker.pkg.dev/solid-antler-409714/gcprepo'
        APP_NAME = 'httpd'
        RELEASE_NAME = 'chereddy'
        
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

        stage("Creating helm charts"){
            steps{
                script{
            sh("sed -i 's/TAG_NAME/${env.BUILD_ID}/g' k8app/values.yaml")
            def CHAR_VER = sh(script: "grep '^version' k8app/Chart.yaml | cut -d ':' -f 2|sed 's/ //g'", returnStdout: true )
			def CHART_NAME = sh(script: "echo k8app-'${CHAR_VER}'.tgz", returnStdout: true )
            sh("helm package k8app")
            sh("helm push '${CHART_NAME}' oci://us-east1-docker.pkg.dev/solid-antler-409714/helmrepo")
            }
        }
        }

        stage('Deploying the app into K8'){
            steps{
                script{
            sh('gcloud container clusters get-credentials friends --zone us-west4-b --project solid-antler-409714')
            sh("helm upgrade '${RELEASE_NAME}' oci://us-east1-docker.pkg.dev/solid-antler-409714/helmrepo/k8app --version '${CHAR_VER}'")
        }
            }
        }
        

            }
            }
        
    
