pipeline {
    agent { label 'master' }
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
		stage('Ansible') {
            steps {
			sh '''
			cd ansible
			ansible-playbook -i inventories/hosts -l linux deploy-package.yml  -e "ansible_user=vagrant" -e "ansible_password=vagrant"
			'''
            }
        }//end of ansible
      }//end stages
    }//end pipeline
