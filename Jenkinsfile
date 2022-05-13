def ansible = [:]
         ansible.name = 'ansible'
         ansible.host = '172.31.8.246'
         ansible.user = 'centos'
         ansible.password = 'Rnstech@123'
         ansible.allowAnyHosts = true
def kops = [:]
         kops.name = 'kops'
         kops.host = '172.31.4.149'
         kops.user = 'centos'
         kops.password = 'Rnstech@123'
         kops.allowAnyHosts = true
pipeline {
    agent { label 'buildserver'}

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven3"
    }

    stages {
        stage('Prepare-Workspace') {
            steps {
                // Get some code from a GitHub repository
                git credentialsId: 'github_credentials', url: 'https://github.com/sathish108/Maven-Java-Project'
		        stash 'Source'
            }
            
        }
        stage('Tools-Setup') {
            steps {
		         echo "Tools Setup"
                 // sshCommand remote: ansible, command: 'cd Maven-Java-Project; git pull'
                 //sshCommand remote: ansible, command: 'cd Maven-Java-Project; ansible-playbook -i hosts tools/sonarqube/sonar-install.yaml'
                 //sshCommand remote: ansible, command: 'cd Maven-Java-Project; ansible-playbook -i hosts tools/docker/docker-install.yml'   
                     
                //K8s Setup
                //sshCommand remote: kops, command: "cd Maven-Java-Project; git pull"
	            //sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/staging/namespace/staging-ns.yml"
	            //sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/prod/namespace/prod-ns.yml"
            }            
        }
	    
    	stage('SonarQube analysis') {
         
            steps{
                echo "Sonar Scanner"
                sh "mvn clean compile"
                withSonarQubeEnv('sonar-7') { 
                sh "mvn sonar:sonar "
                }                     
           }
       }
	    
       stage('Unit Test Cases') {
         
          steps{
	          echo "Clean and Test"
              sh "mvn clean test"  
          }
          post{
              success{
		        echo "Clean and Test"
                  junit 'target/surefire-reports/*.xml'
              }
          }
       }
	    
       stage('Build Code') {
        
          steps{
	          unstash 'Source'
              sh "mvn clean package"  
          }
          post{
              success{
                  archiveArtifacts '**/*.war'
              }
          }
       }
	    
       stage('Build Docker Image') {
          steps{
            sh "docker build -t sathish108/webapp ."  
          }
        }
	    
       stage('Publish Docker Image') {
           steps{

    	       withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
    		    sh "docker login -u ${dockerUser} -p ${dockerPassword}"
	           }
        	   sh "docker push sathish108/webapp"
           }
       }
	    
       stage('Deploy to Staging') {
	
	     steps{
	      //Deploy to K8s Cluster 
              echo "Deploy to Staging Server"
	       sshCommand remote: kops, command: "cd Maven-Java-Project; git pull"
	       sshCommand remote: kops, command: "kubectl delete -f Maven-Java-Project/k8s-code/staging/app/deploy-webapp.yml"
	       sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/staging/app/."
	     }		    
       }
	    
       stage ('Integration-Test') {
	
	        steps {
             echo "Run Integration Test Cases"
             unstash 'Source'
            sh "mvn clean verify"
           }
        }
	    
    
        stage ('Prod-Deploy') {
	
	        steps {
                 echo "Deploy to Production"
	             //Deploy to Prod K8s Cluster
	            sshCommand remote: kops, command: "cd Maven-Java-Project; git pull"
	            sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/prod/app/."
	        }
	    }

    }
}
