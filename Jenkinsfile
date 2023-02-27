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
                    workspaceVolume persistentVolumeClaimWorkspaceVolume(claimName: 'workspace', readOnly: false)

                }
            }

            steps {
                container('maven') {
                    // configFileProvider([configFile(fileId: "${params.MAVEN_SETTINGS_ID}", variable: 'MAVEN_SETTINGS')]) {
                        
                        sh "pwd"
                        sh "id"
                        sh "ls -l /home/jenkins/agent/workspace"
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
                branch 'master'
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
                    sh "ls -l"
                    sh "id"
                    sh "echo $HOME"

                    script {
                        def jobName = env.JOB_NAME
                        def serviceName = jobName.split("/")[0]
                        env.SERVICE_NAME = serviceName
                    }
                                        
                    sh "/kaniko/executor \
                        --context=dir://$HOME/agent/workspace/${SERVICE_NAME}/ \
                        --dockerfile=Dockerfile.jvm  \
                        --destination=gcr.io/paolos-playground-323415/${SERVICE_NAME}:${BUILD_NUMBER} \
                        --destination=gcr.io/paolos-playground-323415/${SERVICE_NAME}:${GIT_COMMIT}"

                }            
            }
        }

        // stage('Update GitOps Repo - Deploy to testing env') {
            
        //     when {
        //         beforeAgent true
        //         branch 'master'
        //     }

        //     options {
        //         skipDefaultCheckout true
        //     }
            
        //     steps {

        //         container('jnlp') { 
        //             sh "pwd"
                    
        //             withCredentials([usernamePassword(credentialsId: 'id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        
        //                 sh "rm -rf your-repo"
        //                 sh "git clone repo" 
        //             }

        //             sh "ls -l"
        //         }

        //         container('jnlp') { 
        //             dir('folder/service') {

        //                 // sh "kustomize edit set image ${SERVICE_NAME}:${BUILD_NUMBER}"

        //                 sh "cat deploy.yaml"
        //                 sh "sed -i 's+cicd/code-with-quarkus.*+cicd/code-with-quarkus:${BUILD_NUMBER}+g' deploy.yaml"
        //                 sh "cat deploy.yaml"                      
        //             }
        //         }

        //         container('jnlp') {
        //             withCredentials([usernamePassword(credentialsId: 'id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

        //                 sh "git config user.email \"jenkins-bot@gmail.com\""
        //                 sh "git config user.name \"Jenkins Bot\""

        //                 sh "git commit -am \"updated app ${SERVICE_NAME} to version ${BUILD_NUMBER}\""
        //                 sh "git push repo.git"             

        //             }
        //         }
        //     }
        // }

    }

    post {

        always {
            sh 'echo post-action-always'

            // node('maven') { 
            //     // sh 'ls -l /tmp/workspace'
            //     // sh 'rm -rf /tmp/workspace/${SERVICE_NAME}'
            // }

            // cleanWs()
        }        
        
    }

}