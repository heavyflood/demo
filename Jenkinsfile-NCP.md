

# Jenkins Pipeline NCP

    node {
    	def groupName = ''
    	def appName = ''
    	def appVersion = ''
    	def systemCode = ''
    	def latestImage = ''
    	def k8sAppName = ''
    	def errorMessage = ''
    	def jarFile = ''
    
    	stage ('Checkout') {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'd0e45a32-5c86-430c-9108-f0e921e72b64', url: 'http://devops-reg.ncp.sicc.co.kr:10080/heavyflood/demo.git']]])
    	}
    	
    	stage('Initialize') {
    		sh "chmod +x ./gradlew"
    		appName = sh(script: "./gradlew properties -q | grep \"name\" | awk '{print \$2}'", returnStdout: true).trim()
    		groupName = sh(script: "./gradlew properties -q | grep  \"group\" | awk '{print \$2}'", returnStdout: true).trim()
    		appVersion = sh(script: "./gradlew properties -q | grep \"version:\" | awk '{print \$2}'", returnStdout: true).trim()
    		k8sAppName = 'app'
    		//latestImage = "devops-reg.ncp.sicc.co.kr/" + k8sAppName + ":" + appVersion + ".${BUILD_NUMBER}"
    		latestImage = "heavyflood/demo" + ":" + appVersion + ".${BUILD_NUMBER}"
    		jarFile = appName + '-' + appVersion + '.jar'
    		echo latestImage
    	}
    
    	stage('Build') {
    		sh "./gradlew build"
    	}
    
    	stage('Archive') {
    		archiveArtifacts artifacts: '**/build/libs/'+ appName + '-' + appVersion +'*.jar', fingerprint: true
    	}
    
    	stage ('Deploy') {
    	    def script = "; envsubst < kubernetesfile.yaml > deployment.yaml"
    	    sh 'export APP_NAME=' + k8sAppName + ' IMAGE=' + latestImage + script
    	    sh 'cat deployment.yaml'
    		sh 'mv build/libs/' + jarFile + ' ./app.jar'
            //sh 'docker container ls -a -f name=app -q | xargs -r docker container stop'
            //sh 'docker container ls -a -f name=app -q | xargs -r docker container rm'
            //sh 'docker rmi -f localhost:5000/app'
            sh 'docker login -u heavyflood -p **0118ghdtn'
            sh 'docker image build -t app . '
            sh 'docker image tag app '+ latestImage
            sh 'docker image push ' + latestImage
            sh 'kubectl apply -n heavyflood -f deployment.yaml'
    		//sh 'kubectl set image -n heavyflood deployment/app app='+ latestImage
    		//sh 'docker run -d -p 9091:9091 --name app '+ latestImage
            echo "finished"
    	}
    }
