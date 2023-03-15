pipeline {

    agent any

    parameters {

        booleanParam(defaultValue: true,description: 'Deploy to Staging Environment ?', name: 'DEPLOY_STAGING')
        
        string(defaultValue: "maven-settings-general", description: 'Maven Settings File Id', name: 'MAVEN_SETTINGS_ID')
        string(defaultValue: "http://my-domain.com/nexus", description: 'Nexus URL', name: 'NEXUS_URL')
        string(defaultValue: "artifacts", description: 'Nexus hosted repo name', name: 'NEXUS_HOSTED_NAME_RELEASES')
        string(defaultValue: "artifacts-snapshots", description: 'Nexus hosted snapshots repo name', name: 'NEXUS_HOSTED_NAME_SNAPSHOTS')
        
        string(defaultValue: 'test@gmail.com', description: 'Notification recipients', name: 'EMAIL_RECIPIENT_LIST')
    }
    
    options {
        
        // use colors in Jenkins console output
        // ansiColor('xterm')
        
        // discard old builds
        buildDiscarder logRotator(daysToKeepStr: '30', numToKeepStr: '45')
        
        // disable concurrent builds and disable build resume
        disableConcurrentBuilds()
        // disableResume()

        // timeout on whole pipeline job
        timeout(time: 1, unit: 'HOURS')
    }
    
    triggers {
        // cron('H */4 * * 1-5')
        pollSCM "*/5 * * * *"      
    }

    environment {

        CI_CD_NAMESPACE = 'jenkins'
        GCP_PROJECT     = "paolos-playground-323415"
        CONTAINER_REG   = "gcr.io"
    }
  
    stages {

        stage("Initialize") {

            options {
                skipDefaultCheckout true
            }

            steps {
                script {
                        echo "Build number: ${BUILD_NUMBER} - Build id: ${env.BUILD_ID} started from Jenkins instance: ${env.JENKINS_URL}"

                        // echo "Deploy to Staging? :: ${params.DEPLOY_STAGING}"

                        echo "Maven Settings file id? :: ${params.MAVEN_SETTINGS_ID}"
                        echo "NEXUS URL? :: ${params.NEXUS_URL}"
                        sh "printenv | sort"

                }
            }
        }

        stage('Code Build and Test') {

            agent { 
                kubernetes {
                    // label 'maven'
                    cloud 'kubernetes'
                    yamlFile 'pod-template-mvn.yaml'
                    workspaceVolume persistentVolumeClaimWorkspaceVolume(claimName: 'workspace', readOnly: false)

                }
            }

            when {
                beforeAgent true
                branch 'helm-test'
            }

            steps {
                container('maven') {
                    // configFileProvider([configFile(fileId: "${params.MAVEN_SETTINGS_ID}", variable: 'MAVEN_SETTINGS')]) {
                        
                        sh "pwd"
                        sh "id"
                        sh "ls -l ${HOME}/agent/workspace"
                        sh "echo $HOME"
                        sh "ls -l /root/.m2"

                        sh "mvn -U -B package"
                    // }
                }       

            }
        }

        stage('Image Build and Push') {

            agent {
                kubernetes {
                    yamlFile 'pod-template-kaniko.yaml'
                    cloud 'kubernetes'
                    workspaceVolume persistentVolumeClaimWorkspaceVolume(claimName: 'workspace', readOnly: false)
                }
            } 

            when {
                beforeAgent true
                branch 'helm-test'
            }

            options {
                skipDefaultCheckout true
            }

            steps {
                container('kaniko') {
                    sh "pwd"
                    sh "ls -l"
                    sh "id"
                    sh "echo $HOME"
                    sh "echo $WORKSPACE"

                    script {
                        def jobName = env.JOB_NAME
                        def serviceName = jobName.split("/")[0]
                        env.SERVICE_NAME = serviceName
                    }
                                        
                    sh "/kaniko/executor \
                        --context=dir://$WORKSPACE --dockerfile=Dockerfile.jvm  \
                        --destination=${CONTAINER_REG}/${GCP_PROJECT}/${SERVICE_NAME}:${BUILD_NUMBER} \
                        --destination=${CONTAINER_REG}/${GCP_PROJECT}/${SERVICE_NAME}:${GIT_COMMIT} \
                        --cache" 

                }            
            }
        }

        stage('Deploy') {

            agent {
                kubernetes {
                    yamlFile 'pod-template-kikd.yaml'
                    cloud 'kubernetes'
                }
            } 

            when {
                beforeAgent true
                branch 'helm-test'
            }

            options {
                skipDefaultCheckout true
            }

            environment {
                GKE_CLUSTER     = "dev-cluster"
                GKE_ZONE        = "asia-east1-a"
                KUBECONFIG      = "${HOME}/agent/.kube"
                NAMESPACE       = "apps"
            }

            steps {
                container('kikd') {
                    sh "pwd"
                    sh "ls -l"
                    sh "id"
                    sh "echo $HOME"

                    // clone manifest repo
                    git credentialsId: 'jenkins-git-ssh-key', url: 'git@github.com:paolocarta/gitops-repo-cicd-course.git'

                    sh "ls -l"

                    // connect to gke cluster
                    sh "gcloud auth activate-service-account jenkins-gcr-push@${GCP_PROJECT}.iam.gserviceaccount.com --key-file=/secret/jenkins-gcr-push-sa-private-key.json --project=$GCP_PROJECT"
                    sh "gcloud container clusters get-credentials $GKE_CLUSTER --zone $GKE_ZONE"
                    
                    sh "ls -la $HOME/agent"  

                    script {
                        def jobName = env.JOB_NAME
                        def serviceName = jobName.split("/")[0]
                        env.SERVICE_NAME = serviceName
                    }
                    
                    sh """
                        cd apps-helm/code-with-quarkus
                        kubectl config view
                        helm template .
                        helm install ${SERVICE_NAME}-helm .
                        helm list
                        kubectl rollout status deployment $SERVICE_NAME -n ${NAMESPACE}
                    """   

                }
            }

        }

        stage('Smoke Test') {

            agent {
                kubernetes {
                    yamlFile 'pod-template-kaniko.yaml'
                    cloud 'kubernetes'
                }
            } 

            when {
                beforeAgent true
                branch 'helm-test'
            }

            options {
                skipDefaultCheckout true
            }

            steps {
                container('kaniko') {
                    sh "pwd"
                    sh "ls -l"
                    sh "id"
                    sh "echo $HOME"
                    sh "echo $WORKSPACE"

                }            
            }
        }


    }

    post {

        always {
            sh 'echo post-action-always'

            // node('maven') { 
            //     // sh 'ls -l /tmp/workspace'
            //     // sh 'rm -rf /tmp/workspace/${SERVICE_NAME}'
            // }

            cleanWs()
        }        
        
    }

}