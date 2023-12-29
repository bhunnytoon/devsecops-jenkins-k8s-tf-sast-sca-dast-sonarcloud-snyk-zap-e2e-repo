pipeline {
  agent any
  tools { 
        maven 'Maven_3_5_2'  
    }
   stages{
    stage('Compile and SAST Scanning') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=natee-securityguru -Dsonar.organization=natee-securityguru -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=6a1187bc7577cd315ff682c540405c5ec144dfaa'
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
                    docker.withRegistry('https://145988340565.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
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
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 
  }
}
