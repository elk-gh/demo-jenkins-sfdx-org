#!groovy
import groovy.json.JsonSlurperClassic
node {
	
    def SF_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def SF_USERNAME=env.HUB_ORG_DH
    def SERVER_KEY_CREDENTIALS_ID=env.JWT_CRED_ID_DH
    def DEPLOYDIR='src'
    def TEST_LEVEL='RunLocalTests'
    def SF_INSTANCE_URL = env.SFDC_HOST_DH ?: "https://test.salesforce.com"


    def toolbelt = tool 'toolbelt'
	
    println 'KEY IS' 
    println SERVER_KEY_CREDENTIALS_ID
    println SF_USERNAME
    println SF_INSTANCE_URL
    println SF_CONSUMER_KEY

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
	
    withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {

		
	// -------------------------------------------------------------------------
	// Authenticate to Salesforce using the server key.
	// -------------------------------------------------------------------------
	stage('Authorize to Salesforce') {
		rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile \"${server_key_file}\" --setdefaultdevhubusername --instanceurl ${SF_INSTANCE_URL}"
	    if (rc != 0) {
		error 'Salesforce org authorization failed.'
	    }
	}
		
		
	// -------------------------------------------------------------------------
	// Deploy metadata and execute unit tests.
	// -------------------------------------------------------------------------
	stage('Deploy and Run Tests') {
	    rc = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy -d manifest/. -u ${SF_USERNAME}"
	    if (rc != 0) {
		error 'Salesforce deploy and test run failed.'
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

