pipeline {

    agent any

    parameters {

        booleanParam(defaultValue: false,description: 'Deploy to Staging Environment ?', name: 'DEPLOY_STAGING')
        
        string(defaultValue: "maven-settings-general", description: 'Maven Settings File Id', name: 'MAVEN_SETTINGS_ID')
        string(defaultValue: "http://ccg.internal.com/nexus", description: 'Nexus URL', name: 'NEXUS_URL')
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
        pollSCM "*/1 * * * *"
    }

    environment {

        CI_CD_NAMESPACE   = 'cicd'
    }
  
    stages {

        stage("Initialize") {

            options {
                skipDefaultCheckout true
            }

            steps {
                script {
                        echo "Build number: ${BUILD_NUMBER} - Build id: ${env.BUILD_ID} started from Jenkins instance: ${env.JENKINS_URL}"

                        echo "Deploy to Staging? :: ${params.DEPLOY_STAGING}"

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

                }
            }

            steps {
                container('maven') {
                    // configFileProvider([configFile(fileId: "${params.MAVEN_SETTINGS_ID}", variable: 'MAVEN_SETTINGS')]) {
                        
                        sh "pwd"
                        sh "id"
                        // sh "ls -l /tmp/workspace"
                        sh "echo $HOME"
                        sh "ls -l /root/.m2"
                        sh "mvn -U -B package"
                    // }
                }       

            }
        }

        stage('Image Build') {

            agent {
                kubernetes {
                    yamlFile 'pod-template-kaniko.yaml'
                    cloud 'kubernetes'
                }
            } 

            when {
                beforeAgent true
                branch 'test'
            }

            options {
                skipDefaultCheckout true
            }

            // environment {
            //     HTTP_PROXY        = ''
            //     HTTPS_PROXY       = ''
            // }

            steps {
                container('kaniko') {
                    sh "pwd"
                    sh "id"
                    sh "echo $HOME"

                    // script {
                    //     def jobName = env.JOB_NAME
                    //     def serviceName = jobName.split("/")[0]
                    //     env.SERVICE_NAME = serviceName
                    // }
                                        
                    sh "/kaniko/executor --dockerfile ./Dockerfile --context . \
                    --destination=gcr.io/paolos-playground-323415/code-with-quarkus:${BUILD_NUMBER} \
                    --destination=gcr.io/paolos-playground-323415/code-with-quarkus:${GIT_COMMIT}"

                }            
            }
        }

        // stage('Image Push') {

        //     agent {
        //         kubernetes {
        //             yamlFile 'pod-template-buildah.yaml'
        //             cloud 'kubernetes'
        //         }
        //     }

        //     options {
        //         skipDefaultCheckout true
        //     }

        //     steps {
        //         container('buildah') {
        //             sh "pwd"
        //             sh "id"

        //             withCredentials([string(credentialsId: 'jenkins-sa-token-power-mi', variable: 'TOKEN')]) {
                        
        //                 sh "buildah --storage-driver=vfs push --tls-verify=false \
        //                     --creds jenkins:${TOKEN} \
        //                     image-registry.openshift-image-registry.svc:5000/${CI_CD_NAMESPACE}/${JOB_NAME}:${BUILD_NUMBER} \
        //                     docker://image-registry.openshift-image-registry.svc:5000/${CI_CD_NAMESPACE}/${JOB_NAME}:${BUILD_NUMBER}"    
        //             }
        //         }            
        //     }
        // }

        stage('Update GitOps Repo') {
            
            // when {
            //     branch 'master'
            // }
            
            steps {
               sh "echo updating gitops repo" 
            }
        }

    }

    post {

        always {
            sh 'echo post-action-always'

            // node('maven') { 
            //     // sh 'ls -l /tmp/workspace'
            //     // sh 'rm -rf /tmp/workspace/code-with-quarkus'
            // }

            // cleanWs()
        }        
        
    }

}