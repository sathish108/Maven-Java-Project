def ansible = [:]
         ansible.name = 'ansible'
         ansible.host = '172.31.42.128'
         ansible.user = 'centos'
         ansible.password = 'Rnstech@123'
         ansible.allowAnyHosts = true
def kops = [:]
         kops.name = 'kops'
         kops.host = '172.31.45.62'
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
                git credentialsId: 'github-server-credentials', url: 'https://github.com/sathish108/Maven-Java-Project.git'    
		stash 'Source'
            }
            
        }
        stage('Tools-Setup') {
            steps {
		    echo "Tools Setup"
                sshCommand remote: ansible, command: 'cd Maven-Java-Project; git pull'
                //sshCommand remote: ansible, command: 'cd Maven-Java-Project; ansible-playbook -i hosts tools/sonarqube/sonar-install.yaml'
                sshCommand remote: ansible, command: 'cd Maven-Java-Project; ansible-playbook -i hosts tools/docker/docker-install.yml'   
                     
                //K8s Setup
                //sshCommand remote: kops, command: "cd Maven-Java-Project; git pull"
	       //sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/staging/namespace/staging-ns.yml"
	       //sshCommand remote: kops, command: "kubectl apply -f Maven-Java-Project/k8s-code/prod/namespace/prod-ns.yml"
            }            
        }
	    
	stage('SonarQube analysis') {
         
          steps{
                echo "Sonar Scanner"
                  //sh "mvn clean compile"
               //withSonarQubeEnv('sonar-7') { 
                 //sh "mvn sonar:sonar "
                }                     
          }
      }
	    
      stage('Unit Test Cases') {
         
          steps{
	       echo "Clean and Test"
              //sh "mvn clean test"  
          }
          post{
              success{
		      echo "Clean and Test"
                  //junit 'target/surefire-reports/*.xml'
              }
          }
      }
	    
      stage('Build Code') {
        
          steps{
		  echo "build code"
	      //unstash 'Source'
              //sh "mvn clean package"  
          }
          post{
              success{
		      echo "archive artifact"
                  //archiveArtifacts '**/*.war'
              }
          }
      }
	    
      stage('Build Docker Image') {
         
         steps{
                  sh "docker build -t sathish108/webapp ."  
         }
      }
	    
  
    }
   }
