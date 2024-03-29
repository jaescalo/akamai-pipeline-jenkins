pipeline {
    agent {
        docker { image 'akamai/shell' }
    }
    environment {
        EDGERC_FILE = credentials('AKAMAI_EDGERC')
        ACCOUNTKEY = '1-6JHGX'
        PIPELINE_NAME = 'gitlab-pipeline-demo'
        ENVIRONMENT = 'prod'
    }
    stages {
        stage('Checkout main branch') {
            steps {
              checkout([$class: 'GitSCM', 
                branches: [[name: '*/main']],
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
                sh 'akamai pipeline merge  -n -v -p $PIPELINE_NAME $ENVIRONMENT --edgerc $EDGERC_FILE --accountSwitchKey $ACCOUNTKEY'
            }
        }

        stage('Update Property') {
            steps {
                echo "$COMMIT_MESSAGE"
                sh 'akamai property-manager property-update -p $ENVIRONMENT.$PIPELINE_NAME --file ./$PIPELINE_NAME/dist/$ENVIRONMENT.$PIPELINE_NAME.papi.json --message "Created By Jenkins-$JOB_BASE_NAME:$BUILD_NUMBER; Commit $COMMIT_MESSAGE" --edgerc $EDGERC_FILE --accountSwitchKey $ACCOUNTKEY'
            }
        }

        stage('Activation') {
            steps {
                sh 'akamai property-manager activate-version -p $ENVIRONMENT.$PIPELINE_NAME --network staging --wait-for-activate --edgerc $EDGERC_FILE --accountSwitchKey $ACCOUNTKEY'
            }
        }

        stage('Test') {
            steps {
                build job: 'Newman-Test-Stage', propagate: true, wait: true, parameters: [
                  string(name: 'ENVIRONMENT', value: env.ENVIRONMENT)
                ]
            }
        }
    }
}