node {
    def app

	def remote = [:]
    remote.name = 'prod-server'
    remote.host = '54.175.228.93'
    remote.user = 'ec2-user'
    remote.password = 'India@321'
    remote.allowAnyHosts = true
    
    stage('Checkout') {
        checkout scm
    }
	
    stage('Maven Package') {
        withMaven(maven:'maven-3.6.3'){
          sh 'mvn clean package'
	}
    }

    stage('Build Image') {
        app = docker.build("learntechpuzz/demo-calculator-service:${env.BUILD_NUMBER}")
    }

    stage('Push Image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
        }
    }
    stage('Deploy Container'){
 		writeFile file: 'deploy.sh', text: "docker stop demo-calculator-service || true && docker rm demo-calculator-service || true && docker run --name demo-calculator-service -d -p 7001:7001 learntechpuzz/demo-calculator-service:$BUILD_NUMBER"
      	sshScript remote: remote, script: "deploy.sh"   
    }
    
    stage('Send Email') {
	    def mailRecipients = "bvijaycom@gmail.com"
		def emailBody = '${SCRIPT, template="groovy-html.template"}'
		def emailSubject = "${env.JOB_NAME} - Build# ${env.BUILD_NUMBER} - ${currentBuild.currentResult}"

	    emailext body: "${emailBody}", mimeType: 'text/html', subject: "${emailSubject}", to: "${mailRecipients}"
	}
}

