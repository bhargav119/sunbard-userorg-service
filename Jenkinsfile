pipeline {
    agent {
        label 'sunbird'
    }
    
    environment {
        JAVA_HOME = "${tool 'JDK'}"
    }

    stages {
        stage('Setup Environment') {
            steps {
                ansiColor('xterm') {
                    script {
                        if (!env.hub_org) {
                            echo "\u001B[1m\u001B[31mUh Oh! Please set a Jenkins environment variable named hub_org with value as registery/sunbidrded\u001B[0m"
                            error 'Please resolve the errors and rerun..'
                        } else {
                            echo "\u001B[1m\u001B[32mFound environment variable named hub_org with value as: ${env.hub_org}\u001B[0m"
                        }
                    }
                }
            }
        }
        
        stage('Checkout Code') {
            steps {
                cleanWs()
                checkout scm
            }
        }
        
        stage('Generate Build Tag') {
            steps {
                script {
                    def commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def build_tag = "${params.github_release_tag.split('/')[-1]}_${commit_hash}_${env.BUILD_NUMBER}"
                    echo "build_tag: ${build_tag}"
                    env.BUILD_TAG = build_tag
                }
            }
        }
        
        stage('Build Project') {
            steps {
                script {
                    env.NODE_ENV = "build"
                    println "Environment will be : ${env.NODE_ENV}"
                    sh 'git log -1'
                    sh "mvn clean install -DskipTests=true -DCLOUD_STORE_GROUP_ID=${params.cloud_store_group_id} -DCLOUD_STORE_ARTIFACT_ID=${params.cloud_store_artifact_id} -DCLOUD_STORE_VERSION=${params.cloud_store_version}"
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                sh "mvn clean install '-Dtest=!%regex[io.opensaber.registry.client.*]' -DfailIfNoTests=false -Dsurefire.failIfNoSpecifiedTests=false -DCLOUD_STORE_GROUP_ID=org.sunbird -DCLOUD_STORE_ARTIFACT_ID=cloud-store-sdk -DCLOUD_STORE_VERSION=1.4.6"
            }
        }
        
        stage('Package Artifacts') {
            steps {
                dir('controller') {
                    sh 'mvn play2:dist'
                }
                sh('chmod 777 ./build.sh')
                sh "./build.sh ${env.BUILD_TAG} ${env.NODE_NAME} ${env.hub_org}"
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts "metadata.json"
                setDescription("${env.BUILD_TAG}")
            }
        }
    }
    
    post {
        failure {
            script {
                currentBuild.result = "FAILURE"
                error "Pipeline failed! Check logs for details."
            }
        }
    }
}
