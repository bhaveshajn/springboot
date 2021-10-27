pipeline {
	agent { label 'master' }
	
	environment {
		vm_creds = credentials('vagrant')
	}
	
	tools {
    	   maven '3.6.3'
	   jdk '1.8'
	}
	stages {
		stage('Build') {
			steps {
				sh 'mvn -B -DskipTests clean package'
				//sh 'mvn deploy -s settings.xml'
			}
		}//end build
		
		stage('Push Package') {
			steps {
				sh 'mvn deploy -s settings.xml'
			}
		}//end push packages
		
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
		
		//stage("Sonar Quality gate") {
		//	steps {
		//		waitForQualityGate abortPipeline: true
		//	}
		//}//end of Sonar Quality gate
		
		stage('Ansible') {
			steps {
				sh '''
				cd ansible
				export ANSIBLE_HOST_KEY_CHECKING=False
				ansible-playbook -i inventories/hosts -l linux deploy-package.yml  -e ansible_user=$vm_creds_USR -e ansible_password=$vm_creds_PSW
				'''
			}
		}//end of ansible
		
	}//end stages
}//end pipeline
