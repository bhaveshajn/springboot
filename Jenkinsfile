pipeline {
	agent any
	environment {
		PROJECT_ID = 'leafy-market-327511'
                CLUSTER_NAME = 'devops'
                LOCATION = 'us-central1-c'
                CREDENTIALS_ID = 'Kubernetes'		
	}
	//{ label 'master' }
	
	//environment {
	//	vm_creds = credentials('vagrant')
	//}
	
	tools {
    	   maven '3.6.3'
	  // jdk '1.8'
	}
	stages {
		stage('Build') {
			steps {
				sh 'mvn -B -DskipTests clean package'
			}
		}//end build
		
		stage('Test') {
			steps {
				sh 'mvn test'
				junit 'target/surefire-reports/*.xml'
			}
		}//end of test
		
		stage('Sonar Analysis') {
			steps {
				withSonarQubeEnv('SonarQube') {
					sh 'mvn sonar:sonar' 
				}
			}
		}//end of sonar
		
		stage("Sonar Quality gate") {
			steps {
				script {
				//waitForQualityGate abortPipeline: true
				def qualitygate = waitForQualityGate()
      				if (qualitygate.status != "OK") {
         			error "Pipeline aborted due to quality gate failure: ${qualitygate.status}"
				}
			   }//end of script
			}
		}//end of Sonar Quality gate
		
		stage('Push Package') {
			steps {
				sh 'mvn deploy -s settings.xml'
			}
		}//end push packages
		
		stage('Docker Build') {
	    steps {
		withDockerRegistry([ credentialsId: "Artifactirytraining", url: "https://trainingdevopscicd.jfrog.io/" ]) {
		sh 'docker build -t "devops:${BUILD_NUMBER}" .'
		sh 'docker tag "devops:${BUILD_NUMBER}" trainingdevopscicd.jfrog.io/default-docker-local/"devops:${BUILD_NUMBER}"'
		}
	    }
	}//end of Docker Build
	    
	stage('Docker Push') {
	    steps {
		withDockerRegistry([ credentialsId: "Artifactirytraining", url: "https://trainingdevopscicd.jfrog.io/" ]) {
		sh 'docker push trainingdevopscicd.jfrog.io/default-docker-local/"devops:${BUILD_NUMBER}"'
		}
	     }
	}//end of Docker Push	
	stage('Deploy to GKE K8s') {
		    steps{
			script {
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_NUMBER}/g' serviceLB.yaml"
			    echo "Start deployment of serviceLB.yaml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'serviceLB.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		       }
	            }
		}//closed Deploy to GKE K8
		
		
		//stage('Ansible') {
		//	steps {
		//		sh '''
		//		cd ansible
		//		export ANSIBLE_HOST_KEY_CHECKING=False
		//		ansible-playbook -i inventories/hosts -l linux deploy-package.yml  -e ansible_user=$vm_creds_USR -e ansible_password=$vm_creds_PSW
		//		'''
		//	}
		//}//end of ansible
	
	}//end stages
}//end pipeline
