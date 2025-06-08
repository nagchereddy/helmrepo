//def ARTIFACT_CREDS 
pipeline{
    agent any
    environment{
        REPO_CREDS = 'github'
        ARTIFACT_CREDS = credentials('jenkins-sa')
        GITHUB_NAME = 'gcprepo'
        GCR_URL = 'us-central1-docker.pkg.dev/steady-circuit-460515-a0/nchereddy'
        APP_NAME = 'aservices'
        RELEASE_NAME = 'chereddy'
        def CHAR_VER = sh(script: "grep '^version' aservices/Chart.yaml | cut -d ':' -f 2|sed 's/ //g'", returnStdout: true ).trim()
		def CHART_NAME = sh(script: "echo '${APP_NAME}'-'${CHAR_VER}'.tgz\n", returnStdout: true ).trim()
        
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
                withCredentials([file(credentialsId: 'jenkins-sa', variable: 'ARTIFACT_CREDS')]) {
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
            sh("sed -i 's/TAG_NAME/${env.BUILD_ID}/g' '${APP_NAME}'/values.yaml")
            
            sh("helm package '${APP_NAME}'")
            sh("helm push '${CHART_NAME}' oci://us-central1-docker.pkg.dev/steady-circuit-460515-a0/nchereddy")
            }
        }
        }

        stage('Deploying the app into K8'){
            steps{
                script{
            sh('gcloud container clusters get-credentials dev-cluster --region us-central1 --project steady-circuit-460515-a0')
            sh(script: "helm upgrade --install '${RELEASE_NAME}' oci://us-central1-docker.pkg.dev/steady-circuit-460515-a0/nchereddy/'${APP_NAME}' --version '${CHAR_VER}'")
        }
            }
        }
        

            }
            }
        
    
