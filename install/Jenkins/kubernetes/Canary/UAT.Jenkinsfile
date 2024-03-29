// Method to extract namespace of kubernetes
def getNameSpace(nsConfigPath){

    def NS=sh (returnStatus: true, script: "grep -i name: " + nsConfigPath + " |head -n1| cut -d':' -f2 &> namespace.txt")
    output = readFile('namespace.txt').trim()

    output
}

// This will deploy kubernetes deployments along with injecting Envoy sidecar to pods deployment
def applyK8ConfigWithIstio(configPath){
    sh (returnStatus: true, script: "${ISTIO_HOME}/bin/istioctl kube-inject -f ${configPath} | kubectl apply -f -")
}

// This will apply definition file through kubectl
def applyK8Config(configPath){
    sh (returnStatus: true, script: "kubectl apply -f ${configPath}")
}

// This will delete all kubernetes units defined in the provided file
def deleteK8Config(configPath) {
    sh (returnStatus: true, script: "kubectl delete -f ${configPath}")
}

// Pipeline starts from here
pipeline {

    agent { label "k8-master" }

    environment {

            appName = 'news-application'

            // Do not change below this line!
            k8configPath = "${WORKSPACE}/install/deploy/kubernetes"
            namespace = getNameSpace("${k8configPath}/namespace-canary.yaml")

            switchTo  = ''
    }

    stages {

		stage('Clone Repo') {
			steps {
				checkout scm
			}
			/*steps {
			    	checkout([$class: 'GitSCM',
					branches: [[name: '/master']],
					doGenerateSubmoduleConfigurations: false,
					extensions: [[$class: 'CleanCheckout']],
					submoduleCfg: [],
					//userRemoteConfigs: [[credentialsId: 'b2ed5912-90f0-40fb-8f98-a35933a55f30', url: 'https://github.com/nisum-inc/POC_Discovery_Server.git']]
					userRemoteConfigs: [[credentialsId: 'b2fd807f-ce8d-4c98-aa74-792f70925e88', url: 'https://github.com/nisum-inc/POC_Discovery_Server.git']]

				])
			}*/
		} // end of stage

        stage('Create Namespace') {
		    steps {
		        script {
                    applyK8Config(env.k8configPath + "/namespace-canary.yaml")
	    	    }
	    	}
		} // end of stage

		stage('Apply Deployment') {
		    /*environment {
                switchTo = currentDeployingValue(env.namespace, "blue", "green", env.appName).toLowerCase()
            }*/

		    steps {

		      script {
                canaryType = "uat"
              }

		        deleteK8Config(env.k8configPath + "/" + env.appName + "-deployment-" + env.namespace + "-"+ canaryType + ".yaml")
                applyK8ConfigWithIstio(env.k8configPath + "/" + env.appName + "-deployment-" + env.namespace + "-" + canaryType + ".yaml")

        	}
        } // end of stage

        stage('Deploy Service Mesh'){
		    steps {
                script {
                    applyK8Config(env.k8configPath + "/" + env.appName + "-servicemesh-" + env.namespace + ".yaml")
                }
            }
        } // end of stage

    } // end of stages

} // end of pipeline
