#!/usr/bin/env groovy

pipeline {
  agent none

  triggers {
    cron('H 9 * * 2') // Runs at least once a week
  }

  options {
    timestamps() // Shows timestamps in build log
    disableConcurrentBuilds()
    parallelsAlwaysFailFast()
    // You can either disable concurrent builds or use the lock function to run only one deployment at a time, but support concurrent builds.
    buildDiscarder(logRotator(numToKeepStr: '90', artifactNumToKeepStr: '90'))
    /*
        daysToKeepStr: history is only kept up to this days.
        numToKeepStr: only this number of build logs are kept.
        artifactDaysToKeepStr: artifacts are only kept up to this days.
        artifactNumToKeepStr: only this number of builds have their artifacts kept.
    */
  }

  environment {
    AWS_DEFAULT_REGION='eu-west-1'
    FAST_HTTPAUTH=getFastHttpAuth()
    FAST_EMAIL=getFastEmail()
    INVOKED_BUILD_NUMBER=getInvokedBuildNumber()
  }

  stages {

    stage('Run tests and deploy'){
      agent { label 'build-fullstack'}
      when {
        beforeAgent true
        branch 'master'
      }
      steps {
        sh 'npm install'
        sh 'npm run test'
        sh 'npm run build'
        sh 'cf sync cfn/cfn-config.yml --confirm'
        sh 'aws s3 sync dist/ s3://relocation-swagger-ui/ --delete'
        sh 'aws s3 sync dist/ s3://relocation-partner-swagger-ui/ --delete'

      }
    }
  }
}
