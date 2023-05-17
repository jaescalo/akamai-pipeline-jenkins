pipeline {
    agent {
        docker { image 'akamai/shell' }
    }
    environment {
        EDGERC_FILE = credentials('AKAMAI_EDGERC_FILE')
        PIPELINE_NAME = 'demo.pipeline'
    }
    stages {
        stage('Checkout main branch') {
            steps {
              checkout([$class: 'GitSCM', 
                branches: [[name: '*/simplified']],
                doGenerateSubmoduleConfigurations: false,
                extensions: [[$class: 'CleanCheckout']],
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/jaescalo/akamai-pipeline-jenkins.git']]])
              script {
                COMMIT_MESSAGE = sh (script: "git log --oneline -n 1 --pretty=format:'%h'", returnStdout: true).trim()
              }
          }
        }
        
        stage('Build Rule Tree') {
            steps {
                sh 'akamai pipeline merge --no-validate --verbose --pipeline . $ENVIRONMENT --edgerc $EDGERC_FILE'
            }
        }

        stage('Update Property') {
            steps {
                echo "$COMMIT_MESSAGE"
                sh 'akamai property-manager property-update --property $ENVIRONMENT.$PIPELINE_NAME --file ./dist/$ENVIRONMENT...papi.json --message "Created By Jenkins-$JOB_BASE_NAME:$BUILD_NUMBER; Commit $COMMIT_MESSAGE" --edgerc $EDGERC_FILE'
            }
        }

        stage('Activation') {
            steps {
                sh 'akamai property-manager activate-version --property $ENVIRONMENT.$PIPELINE_NAME --network staging --wait-for-activate --edgerc $EDGERC_FILE'
            }
        }
    }
}