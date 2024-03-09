pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   stages{
    stage('Compile and SAST Scanning') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=bhunnytoon -Dsonar.organization=bhunnytoon -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=f478dceb39e2fb62ce2622216a3112a7186cb2e1'
			}
    }

	stage('SCA Scanning') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build Image') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('Push Image') {
            steps {
                script{
                    docker.withRegistry('https://211125298331.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Deploy Buggy Web on Kube Cluster') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}
	   
	stage ('Wait for deploying'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on Kube Cluster"'
	   	}
	   }
	   
	stage('DAST Scanning') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 
  }
}
