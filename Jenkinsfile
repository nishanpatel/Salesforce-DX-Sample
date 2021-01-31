#!groovy

import groovy.json.JsonSlurperClassic

node {
  def SF_CONSUMER_KEY = env.SF_CONSUMER_KEY
  def SF_USERNAME = env.SF_USERNAME
  def SERVER_KEY_CREDENTALS_ID = env.SERVER_KEY_CREDENTALS_ID
  def TEST_LEVEL = env.TEST_LEVEL ?: 'RunLocalTests'
  def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"

  def toolbelt = tool 'toolbelt'

  println("SF_CONSUMER_KEY ${SF_CONSUMER_KEY}")
  println("SF_USERNAME ${SF_USERNAME}")
  println("SERVER_KEY_CREDENTALS_ID ${SERVER_KEY_CREDENTALS_ID}")
  println("TEST_LEVEL ${TEST_LEVEL}")
  println("SF_INSTANCE_URL ${SF_INSTANCE_URL}")

  // -------------------------------------------------------------------------
  // Check out code from source control.
  // -------------------------------------------------------------------------
  stage('checkout source') {
    checkout scm
  }

  // -------------------------------------------------------------------------
  // Run all the enclosed stages with access to the Salesforce
  // JWT key credentials.
  // -------------------------------------------------------------------------
  withEnv(["HOME=${env.WORKSPACE}"]) {
    withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {
      
      // -------------------------------------------------------------------------
      // Authorize the Dev Hub org with JWT key and give it an alias.
      // 
      stage('Authorize DevHub') {
        rc = command "${toolbelt}/sfdx force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername"
        if (rc != 0) {
          error 'Salesforce dev hub org authorization failed.'
        }
      }

      // -------------------------------------------------------------------------
      // Push source to DevHub org.
      // -------------------------------------------------------------------------
      stage('Push To Test Scratch Org') {
        rc = command "${toolbelt}/sfdx force:source:push --targetusername ${SF_USERNAME}"
        if (rc != 0) {
          error 'Salesforce push to test scratch org failed.'
        }
      }

    }
  }

}

def command(script) {
  if (isUnix()) {
    return sh(returnStatus: true, script: script);
  } else {
    return bat(returnStatus: true, script: script);
  }
}